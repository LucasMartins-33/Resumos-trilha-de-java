# Capítulo 04: Desenvolvimento Microserviço de Proposta

Este documento contém o resumo do **Capítulo 04**, focado no desenvolvimento do microserviço de **Proposta** (Proposal). Este microserviço expõe uma API REST para que os "Clientes" enviem propostas de compra de minério de ferro. Ele armazena os dados completos em seu próprio banco de dados (PostgreSQL) e emite uma versão simplificada da proposta para um tópico do Apache Kafka, para que outros serviços (como o de Relatórios) possam consumi-la.

---

## 1. Setup Inicial e Estrutura do Projeto

### Criação (Quarkus Initializer)
Gerado através do [code.quarkus.io](https://code.quarkus.io):
- **Group:** `org.br.mineradora`
- **Artifact:** `proposta`
- ⚠️ **Lembrete de Atualização:** Apesar de o curso usar Java 11, em projetos atuais (Quarkus 3+) utilize sempre **Java 17** ou superior.

**Dependências base selecionadas:**
- RESTEasy Reactive Jackson (Para construir a API REST e trabalhar com JSON)
- Hibernate ORM with Panache (Persistência)
- JDBC Driver - PostgreSQL (Banco de Dados)
- SmallRye Reactive Messaging - Kafka (Mensageria)
- Lombok (Adicionado manualmente no `pom.xml`)

### Estrutura de Pacotes (Arquitetura)
O instrutor reforça o padrão **MVC (Model-View-Controller)** adaptado para APIs REST, separando bem as responsabilidades para evitar que o *Controller* acesse o *Repository* diretamente:
- `controller`: Recebe as requisições HTTP (API REST).
- `service`: Contém as regras de negócio e validações (Intermediário).
- `repository`: Comunica-se com o banco de dados.
- `entity`: Modelos do banco de dados (Tabelas).
- `dto`: *Data Transfer Objects* (tráfego de informações).
- `message`: Emissão de eventos para o Kafka.

---

## 2. Implementação das Camadas

### 2.1. Entidades e DTOs
**`ProposalEntity`**: Representa a tabela `proposal` no banco de dados.
- Os atributos utilizam notações do JPA (`@Id`, `@GeneratedValue`) e as anotações do Lombok (`@Data`, `@NoArgsConstructor`).
- Padrão de Nomenclatura: Foi utilizado o `@Column(name="price_tonne")` para garantir que o *camelCase* do Java (ex: `priceTonne`) seja mapeado corretamente para o *snake_case* esperado pelo banco (ex: `price_tonne`).
- Campos principais: `id`, `customer`, `priceTonne`, `tonnes`, `country`, `proposalValidityDays`, `created`.
> ⚠️ **Atualização Quarkus 3:** Lembre-se que todas as anotações de banco (JPA) e validações agora pertencem aos pacotes `jakarta.*` (ex: `jakarta.persistence.Entity`) em substituição ao antigo `javax.*`.

**DTOs**: Foram criados dois DTOs, usando o padrão `@Builder` e `@Jacksonized` (para converter JSON):
- `ProposalDetailsDTO`: Contém **todos** os campos detalhados da proposta. Usado na comunicação entre a API REST (`Controller`), a camada `Service` e o cliente externo.
- `ProposalDTO`: Contém apenas os campos essenciais (`proposalId`, `customer`, `priceTonne`). É este objeto simplificado que é enviado ao Apache Kafka para não sobrecarregar o fluxo de mensagens.

### 2.2. Acesso a Dados (Repository)
A classe `ProposalRepository` implementa o `PanacheRepository<ProposalEntity>`, herdando os métodos base. O instrutor também demonstrou como criar queries personalizadas, como:
```java
public Optional<ProposalEntity> findByCustomer(String customer) {
    return find("customer", customer).firstResultOptional();
}
```

### 2.3. Mensageria (KafkaEvents)
Assim como no microserviço de cotação, utilizamos o **SmallRye Kafka** com um `@Channel` e um `Emitter<ProposalDTO>` para enviar os dados reduzidos ao tópico do Kafka sempre que uma proposta é criada.

### 2.4. Regras de Negócio (Service)
Foi utilizada uma abordagem baseada em **Interfaces**:
- Criou-se a interface `ProposalService`.
- Criou-se a implementação `ProposalServiceImpl`.
- **Atenção (Bugfix abordado na aula):** A classe de implementação **deve** ser anotada com `@ApplicationScoped`. Se esquecer esta anotação, o Quarkus não conseguirá injetá-la no Controller e a aplicação vai falhar ao subir (erro *Unsatisfied dependency*).

**Métodos principais:**
- `findFullProposal(long id)`: Busca detalhes de uma proposta específica.
- `createNewProposal(ProposalDetailsDTO)`: Salva no DB através do Repository e invoca o `KafkaEvents` para enviar ao tópico. Anotado com **`@Transactional`** por modificar estado no banco.
- `removeProposal(long id)`: Deleta uma proposta pelo ID. Anotado com **`@Transactional`**.

### 2.5. Expondo a API (Controller)
A classe `ProposalController` é anotada com `@Path("/api/proposal")` e expõe três endpoints usando os verbos HTTP correspondentes:
- `@GET @Path("/{id}")`: Retorna os detalhes da proposta em formato JSON.
- `@POST`: Recebe um JSON no corpo da requisição e cria a proposta. Retorna um objeto `Response` indicando sucesso (`200 OK`) ou erro.
- `@DELETE @Path("/{id}")`: Remove a proposta.

O Controller **apenas** recebe a requisição HTTP e delega a operação para o `ProposalService`, mantendo a responsabilidade isolada.

---

## 3. Configurações (`application.properties`)
Configurações essenciais do microserviço:

**Porta da Aplicação:**
```properties
quarkus.http.port=8091
```

**Banco de Dados (PostgreSQL):**
Conecta a um novo banco de dados (`proposal_db`) que deve ser criado manualmente via DBeaver (ou linha de comando) no servidor PostgreSQL existente.
```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.username=postgres
quarkus.datasource.password=1234
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/proposal_db
quarkus.hibernate-orm.database.generation=update
```

**Kafka:**
```properties
mp.messaging.outgoing.proposal-channel.connector=smallrye-kafka
mp.messaging.outgoing.proposal-channel.topic=proposals
kafka.bootstrap.servers=localhost:9092
```

---

## 4. Testes Locais
Foi utilizado o **Postman** para testar a rota POST.
- **URL:** `http://localhost:8091/api/proposal`
- **Body (JSON):**
```json
{
    "customer": "Mr. America",
    "priceTonne": 125.25,
    "tonnes": 800,
    "country": "USA",
    "proposalValidityDays": 3
}
```
**Resultado Esperado:** 
1. Retorna status `200 OK`.
2. Os logs mostram o registro no banco e o envio para o Kafka.
3. No DBeaver, a tabela `proposal` deve aparecer criada (automagicamente pelo Hibernate) com a linha preenchida, incluindo o ID gerado automaticamente.
