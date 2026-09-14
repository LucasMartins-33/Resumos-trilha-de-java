# Capítulo 03: Desenvolvendo Microserviço de Cotação de Moeda

Este documento contém o resumo do **Capítulo 03**, detalhando a implementação prática do primeiro microserviço da nossa arquitetura: o **Microserviço de Cotação**. Ele é responsável por consultar uma API externa, salvar a cotação no banco de dados e enviá-la para o Apache Kafka.

---

## 1. Setup Inicial e Estrutura de Pacotes

### Criando o Projeto (Quarkus Initializer)
A geração do projeto é feita acessando [code.quarkus.io](https://code.quarkus.io) (o "Spring Initializr" do ecossistema Quarkus).
- **Group:** `org.br.mineradora`
- **Artifact:** `cotacao`
- ⚠️ **Atualização Importante:** O instrutor usa Java 11. No ecossistema atual (Quarkus 3+), você deve selecionar **Java 17** ou superior.

**Dependências base selecionadas:**
- Hibernate ORM with Panache (Para comunicação com o banco de dados)
- JDBC Driver - PostgreSQL
- SmallRye Reactive Messaging - Kafka (Mensageria)
- Rest Client Reactive Jackson (Para consumir a API Externa)
- Lombok (Adicionado no `pom.xml` para reduzir código boilerplate)
- Quartz (Para agendamento de tarefas)

### Estrutura de Pacotes
Dentro do pacote raiz `org.br.mineradora`, criamos a seguinte separação de responsabilidades (Clean Architecture base):
- `client`: Comunicação com APIs externas.
- `dto`: *Data Transfer Objects* (objetos para transporte de dados, sem ligação com o banco).
- `entity`: Classes que representam as tabelas do banco de dados.
- `message`: Integração com o Apache Kafka.
- `repository`: Interação com o banco de dados.
- `scheduler`: Agendamento de tarefas (Cron/Timers).
- `service`: Regras de negócio.

---

## 2. Implementação das Camadas

### 2.1. Entidades e Banco de Dados (PostgreSQL)
A classe `QuotationEntity` mapeia a tabela no banco de dados. Utilizamos as anotações do JPA e do Lombok:
- `@Entity` e `@Table(name="quotation")`: Identificam a classe como entidade do banco.
- `@Id` e `@GeneratedValue`: Para gerar o ID automaticamente.
- `@Column(name="currency_price")`: O PostgreSQL tem como padrão usar *snake_case* nas colunas. Usar a anotação garante que o Java (que usa *camelCase*) faça o *match* correto.
> ⚠️ **Atualização Quarkus 3:** As anotações do JPA e do REST (`@Entity`, `@Id`, `@GET`, `@Path`, etc.) agora pertencem aos pacotes `jakarta.*` em vez de `javax.*`.

Para facilitar consultas, a classe `QuotationRepository` implementa `PanacheRepository<QuotationEntity>`. O Panache já fornece nativamente os métodos `.persist()`, `.findAll()`, etc. Ao usar `@ApplicationScoped`, a classe entra no escopo de Injeção de Dependência do Quarkus.

### 2.2. Consumindo API Externa (REST Client)
A aplicação consome a *AwesomeAPI* para buscar o Dólar em tempo real.
A interface `CurrencyPriceClient` utiliza o MicroProfile Rest Client:
```java
@RegisterRestClient(baseUri = "https://economia.awesomeapi.com.br")
@Path("/last")
@ApplicationScoped
public interface CurrencyPriceClient {
    @GET
    @Path("/{pair}")
    CurrencyPriceDTO getPriceByPair(@PathParam("pair") String pair);
}
```
**Nota sobre o JSON:** Para facilitar a conversão (deserialização) da resposta JSON da API para o Objeto Java (DTO), utiliza-se a anotação `@Jacksonized` (junto com o Lombok `@Builder` e `@Data`) na classe DTO.

### 2.3. Mensageria com Apache Kafka
Para enviar a cotação para os outros microserviços, implementou-se a classe `KafkaEvents`.
Utilizamos o **SmallRye Reactive Messaging**.
```java
@ApplicationScoped
public class KafkaEvents {
    @Channel("quotation-channel") // Nome do canal que será configurado no properties
    Emitter<QuotationDTO> quotationRequestEmitter;

    public void sendNewKafkaEvent(QuotationDTO quotation) {
        quotationRequestEmitter.send(quotation).toCompletableFuture().join();
    }
}
```
*Atualização:* A ferramenta recomendada pelo instrutor para gerenciar o Kafka visualmente foi o **Conduktor** (Desktop trial). 

### 2.4. Regra de Negócio (Service)
Na classe `QuotationService` está o coração da regra de negócio:
1. Faz a requisição pelo `CurrencyPriceClient`.
2. Verifica no `QuotationRepository` a última cotação salva.
3. Se a lista estiver vazia **ou** se o preço atual do Dólar for **MAIOR** que o salvo anteriormente:
   - Persiste a nova cotação no banco.
   - Envia um evento com o novo valor para o Apache Kafka usando o `KafkaEvents`.

### 2.5. Agendamento de Tarefas (Scheduler)
Para que a cotação seja verificada automaticamente a cada 35 segundos (requisito do negócio), a classe `QuotationScheduler` utiliza o Quarkus Quartz:
```java
@ApplicationScoped
public class QuotationScheduler {
    @Inject
    QuotationService quotationService;

    @Transactional
    @Scheduled(every = "35s") // A cada 35 segundos
    void schedule() {
        quotationService.getCurrencyPrice();
    }
}
```
A anotação `@Transactional` é vital aqui, pois o Service faz inserções no banco de dados, o que exige uma transação ativa.

---

## 3. Configurações (`application.properties`)
Para amarrar o funcionamento, configuramos o arquivo `application.properties`:

**Banco de Dados:**
```properties
quarkus.datasource.db-kind=postgresql
quarkus.datasource.username=postgres
quarkus.datasource.password=1234
quarkus.datasource.jdbc.url=jdbc:postgresql://localhost:5432/quotation_db
quarkus.hibernate-orm.database.generation=update
```
*(Nota: O banco postgres local foi subido usando o Docker com o comando: `docker run --name postgres -p 5432:5432 -e POSTGRES_PASSWORD=1234 -e POSTGRES_USER=postgres -d postgres`)*

**Kafka:**
```properties
# Emissor do SmallRye apontando para o Tópico 'quotations'
mp.messaging.outgoing.quotation-channel.connector=smallrye-kafka
mp.messaging.outgoing.quotation-channel.topic=quotations
kafka.bootstrap.servers=localhost:9092
```

## 4. Rodando o Projeto
Para iniciar o microserviço em modo de desenvolvimento (Live Reload), usamos:
```bash
mvn quarkus:dev
```
Se tudo estiver correto, a cada 35 segundos o console imprimirá a validação e, caso a cotação mude, a persistência no banco e o envio do DTO para o Kafka ocorrerão automaticamente.
