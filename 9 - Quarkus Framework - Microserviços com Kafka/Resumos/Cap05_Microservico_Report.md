# Capítulo 05: Desenvolvendo Microserviço de Relatório (Report)

Este documento contém o resumo do **Capítulo 05**, focado no desenvolvimento do microserviço de **Report** (Relatórios). Diferente dos microsserviços anteriores que *enviam* dados, a principal função deste serviço é **consumir** as mensagens do Apache Kafka (Cotações e Propostas), cruzar essas informações no seu banco de dados e gerar um arquivo **CSV** para download.

---

## 1. Setup Inicial e Novas Dependências

### Criação (Quarkus Initializer)
Gerado através do [code.quarkus.io](https://code.quarkus.io):
- **Group:** `org.br.mineradora`
- **Artifact:** `report`
- ⚠️ **Lembrete de Atualização:** Utilizar **Java 17** ou superior.

**Dependências base selecionadas:**
- RESTEasy Reactive Jackson (Para a API REST)
- Hibernate ORM with Panache (Persistência)
- JDBC Driver - PostgreSQL (Banco de Dados)
- SmallRye Reactive Messaging - Kafka (Mensageria)
- Lombok
- **Apache Commons CSV:** (Adicionada manualmente). Biblioteca clássica do Java para facilitar a escrita e leitura de arquivos `.csv`.

### Estrutura de Pacotes
A mesma base do padrão MVC (Controller, Service, Repository, Entity, DTO, Message), mas com uma novidade:
- **`utils`**: Pacote criado para classes utilitárias ("Helpers"), ou seja, códigos soltos que não fazem parte estrita da regra de negócio de banco de dados/API, mas ajudam o sistema. Ex: O gerador de CSV.

---

## 2. Implementação das Camadas

### 2.1. Entidades e DTOs
Diferente dos outros serviços, este microserviço possui **duas entidades** de banco de dados, pois ele precisa espelhar os dados que vêm de fora:
1. **`QuotationEntity`**: Para salvar os preços do dólar recebidos via Kafka.
2. **`OpportunityEntity`**: Para salvar a oportunidade real cruzada (Dados da Proposta + Último Preço do Dólar).

**DTOs**:
- `ProposalDTO` e `QuotationDTO`: Para conseguir deserializar os objetos lidos do Apache Kafka.
- `OpportunityDTO`: Para tráfego interno das informações cruzadas.

### 2.2. Acesso a Dados (Repository)
Foram criadas `OpportunityRepository` e `QuotationRepository`, ambas estendendo `PanacheRepository`.

### 2.3. Mensageria - Consumindo do Kafka (`KafkaEvents`)
A grande diferença aqui está nas anotações. Ao invés de enviar (Outgoing), nós vamos **ler (Incoming)** dos tópicos.
```java
@ApplicationScoped
public class KafkaEvents {

    @Inject
    OpportunityService opportunityService;

    @Incoming("proposal-channel")
    @Transactional // IMPORTANTE: Abre a transação para salvar no banco!
    public void receiveProposal(ProposalDTO proposal) {
        opportunityService.buildOpportunity(proposal);
    }

    @Incoming("quotation-channel")
    @Transactional
    public void receiveQuotation(QuotationDTO quotation) {
        opportunityService.saveQuotation(quotation);
    }
}
```
**Atenção (Bugfix abordado na aula):** O instrutor demonstrou que, se esquecer do `@Transactional` acima dos métodos `@Incoming`, o Quarkus lança uma exceção `Transaction is not active` porque o serviço tenta salvar a informação no banco de dados, mas nenhuma transação foi aberta pelo listener do Kafka.

### 2.4. Regras de Negócio e Geração de CSV (Service e Utils)
No `OpportunityServiceImpl`, ocorre o cruzamento de dados:
- Quando uma nova proposta chega via Kafka, o sistema faz um *find* na tabela de Cotações, inverte a lista (`Collections.reverse()`) para pegar a **última cotação inserida (índice 0)**.
- Com os dados da proposta + a última cotação do dólar, ele cria um `OpportunityEntity` e salva no banco de dados.

Na classe **`CSVHelper`** (dentro do pacote `utils`):
- Transforma a lista de `OpportunityDTO` em um `ByteArrayInputStream`.
- Define o cabeçalho do arquivo: `ID Proposta`, `Cliente`, `Preço por Tonelada`, `Melhor cotação de Moeda`.

### 2.5. Expondo o Download via API (Controller)
O `OpportunityController` expõe o endpoint para baixar o relatório. O truque aqui é alterar o *Media Type* retornado.
```java
@GET
@Path("/report")
@Produces(MediaType.APPLICATION_OCTET_STREAM)
public Response generateReport() {
    try {
        return Response.ok(opportunityService.generateCSVOpportunityReport(), MediaType.APPLICATION_OCTET_STREAM)
            .header("Content-Disposition", "attachment; filename=\"oportunidades_venda.csv\"")
            .build();
    } catch (Exception e) {
        return Response.serverError().build();
    }
}
```

---

## 3. Configurações (`application.properties`)

- **Porta:** `8081` (`quarkus.http.port=8081`)
- **Banco de Dados:** Conecta a um novo banco `report_db` (`localhost:5432/report_db`).
- **Kafka:** Configurado para **Incoming**:
```properties
mp.messaging.incoming.proposal-channel.connector=smallrye-kafka
mp.messaging.incoming.proposal-channel.topic=proposals
mp.messaging.incoming.quotation-channel.connector=smallrye-kafka
mp.messaging.incoming.quotation-channel.topic=quotations
```

---

## 4. Testes de Resiliência (A Força dos Microsserviços e do Kafka)
O instrutor demonstrou na prática o motivo de usar essa arquitetura:

1. **Testando falhas no Serviço de Cotação:**
   - Ele derrubou (desligou) o microserviço de cotação.
   - Mesmo assim, ao pedir o relatório no Serviço de Report, ele baixou com sucesso usando a *última cotação de dólar salva no banco* antes da queda. A aplicação não quebra para o usuário final.
   
2. **Testando falhas no Serviço de Report (Consumidor offline):**
   - Ele desligou o microserviço de Report e inseriu uma nova proposta (via Postman) no serviço de Propostas.
   - O serviço de propostas enviou a mensagem para o tópico do Kafka. Como o Report estava offline, a mensagem ficou guardada no broker.
   - Ao religar o microserviço de Report, ele **automaticamente** leu a mensagem pendente no tópico, salvou no seu banco e, ao baixar o relatório novamente, a nova proposta estava lá!

### O Problema Atual: Segurança
No final da aula, o instrutor fez um alerta essencial: **Nossas APIs estão totalmente abertas para a Internet.** Qualquer um pode dar um POST e criar propostas ou baixar relatórios.
O próximo passo lógico (Próximo Capítulo) será bloquear o acesso direto a essas APIs utilizando o padrão de **API Gateway (BFF)** e um servidor de Autenticação/Autorização (**Keycloak**).
