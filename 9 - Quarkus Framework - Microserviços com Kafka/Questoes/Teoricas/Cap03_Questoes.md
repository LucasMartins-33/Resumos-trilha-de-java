# Questões - Capítulo 03: Microsserviço de Cotação

### Questão 1: Qual extensão do Quarkus é utilizada para o mapeamento objeto-relacional (ORM) e persistência de dados no banco?
> [!success]- Resposta
> Hibernate ORM with Panache.

### Questão 2: Em Quarkus/Jakarta, qual anotação indica que um método deve ser executado repetidamente em intervalos de tempo pré-definidos?
> [!success]- Resposta
> `@Scheduled` (do framework Quartz / Quarkus Scheduler). Exemplo: `@Scheduled(every = "35s")`.

### Questão 3: Para consumir uma API REST externa (AwesomeAPI), qual tecnologia do ecossistema Quarkus foi utilizada no lugar de criar clientes HTTP manualmente?
> [!success]- Resposta
> MicroProfile Rest Client (através da anotação `@RegisterRestClient`).

### Questão 4: Na regra de negócio do microserviço de Cotação, quando um evento contendo o preço do dólar é disparado no Kafka?
> [!success]- Resposta
> O evento só é disparado se a cotação consultada na API externa for estritamente **maior** do que a última cotação que estava salva no banco de dados.

### Questão 5: O que acontece se utilizarmos a anotação `@Column(name="currency_price")` numa propriedade Java chamada `currencyPrice`?
> [!success]- Resposta
> A anotação orienta o JPA (Hibernate) a mapear a propriedade `currencyPrice` (camelCase) para a coluna `currency_price` (snake_case) no banco de dados, respeitando o padrão adotado no PostgreSQL.

### Questão 6: Por que o método executado pelo `@Scheduled` na classe `QuotationScheduler` foi anotado com `@Transactional`?
> [!success]- Resposta
> Porque dentro desse método ocorre a chamada ao `QuotationService` que realiza um `.persist()` no banco de dados. Qualquer operação de gravação ou alteração via Hibernate exige uma transação ativa no Quarkus.

### Questão 7: Como o Quarkus sabe para qual servidor Kafka enviar a mensagem e em qual tópico?
> [!success]- Resposta
> Através do mapeamento no arquivo `application.properties`. A chave `mp.messaging.outgoing.[canal].topic` define o nome do tópico, e `kafka.bootstrap.servers` define o endereço do servidor Kafka.

### Questão 8: Qual é a finalidade da anotação `@Jacksonized` (junto com o `@Builder` do Lombok) na classe `CurrencyPriceDTO`?
> [!success]- Resposta
> Ela permite que a biblioteca Jackson utilize o padrão Builder do Lombok para instanciar a classe ao desserializar (converter) o JSON vindo da API externa para o objeto Java automaticamente.

### Questão 9: Em vez de usar os antigos pacotes `javax.persistence.*`, qual deve ser a importação correta para `@Entity` e `@Table` no Quarkus 3 moderno?
> [!success]- Resposta
> Deve-se usar o pacote `jakarta.persistence.*`, devido à transição da especificação Java EE para Jakarta EE.

### Questão 10: Qual interface do Panache o `QuotationRepository` deve implementar para herdar operações prontas como `persist` e `findAll`?
> [!success]- Resposta
> A interface `PanacheRepository<Entidade>` (ex: `PanacheRepository<QuotationEntity>`).
