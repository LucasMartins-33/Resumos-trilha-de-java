📘 Capítulo 03: Desenvolvendo Microserviço de Cotação de Moeda

O Cenário:
O primeiro componente técnico do sistema é o Microserviço de Cotação. Ele deve consultar automaticamente uma API pública a cada 35 segundos, salvar dados no PostgreSQL e enviar mensagens para o Kafka.

Sua missão é validar as implementações de código e a configuração desse microserviço no Quarkus:

🟢 Atividade 3.1: Configuração das Anotações JPA e Jakarta

No Quarkus 3+, as anotações do JPA mudaram de pacote.
Crie um pseudo-código de uma entidade `QuotationEntity` e escreva o nome dos pacotes (ex: `jakarta.persistence.*`) para as anotações `@Entity` e `@Id`.

🟢 Atividade 3.2: Padrão de Nomenclatura no Banco de Dados

O banco de dados PostgreSQL utiliza _snake_case_, mas o Java utiliza _camelCase_.
Como você anota o atributo `currencyPrice` na entidade para garantir que ele seja mapeado para a coluna `currency_price`?

🟢 Atividade 3.3: O Padrão PanacheRepository

O Quarkus simplifica o acesso ao banco com o Panache.
Escreva uma classe `QuotationRepository` que herda as operações básicas de banco. Qual interface ela deve estender?

🟢 Atividade 3.4: Rest Client Reactive

O microserviço consome uma API externa.
No MicroProfile Rest Client, qual anotação é utilizada na interface `CurrencyPriceClient` para definir a URL base (ex: `https://economia.awesomeapi.com.br`)?

🟢 Atividade 3.5: Emissão de Mensagens com SmallRye Kafka

Para enviar um dado ao Kafka, utilizamos o SmallRye Reactive Messaging.
Qual anotação é utilizada acima do atributo `Emitter<QuotationDTO>` para referenciar o canal configurado no `application.properties`?

🟢 Atividade 3.6: A Lógica de Atualização da Cotação

O `QuotationService` verifica a última cotação salva.
Descreva a regra de negócio exata implementada: Quais são as duas condições necessárias para que uma nova cotação seja salva e enviada ao Kafka?

🟢 Atividade 3.7: Agendamento de Tarefas com Quartz

O método que busca a cotação deve rodar a cada 35 segundos.
Como você anota o método `schedule()` utilizando o agendador de tarefas do Quarkus para atingir esse objetivo?

🟢 Atividade 3.8: Transações de Banco de Dados

Dentro da classe agendadora, foi utilizada a anotação `@Transactional`.
Por que essa anotação é vital e o que aconteceria se ela não fosse colocada, sabendo que a operação envolve um `persist()` no banco?

🟢 Atividade 3.9: Configurações do Banco no application.properties

Para o banco rodar localmente, o `application.properties` precisa saber qual tecnologia usar.
Qual é a chave de propriedade que define o tipo do banco de dados (ex: `postgresql`) no Quarkus?

🟢 Atividade 3.10: Configurações de Tópico do Kafka

No `application.properties`, o canal do Kafka (ex: `quotation-channel`) precisa ser roteado para o servidor.
Escreva a propriedade de configuração responsável por definir os endereços do broker (ex: `localhost:9092`).
