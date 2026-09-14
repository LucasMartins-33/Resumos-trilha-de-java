📘 Capítulo 05: Desenvolvendo Microserviço de Relatório (Report)

O Cenário:
O Microserviço de Relatórios atua como o consumidor da nossa arquitetura. Ele lê dos tópicos do Kafka, cruza dados de cotações e propostas em tempo real e gera um arquivo CSV consolidado para os gerentes e operadores.

Sua missão é testar a implementação do consumo assíncrono e cruzamento de informações no Quarkus:

🟢 Atividade 5.1: O Papel de um Consumidor Kafka

Diferente dos microsserviços anteriores, o Report foca em consumir.
Quais são as duas informações externas vitais que ele precisa consumir para conseguir gerar os relatórios? 

🟢 Atividade 5.2: Estrutura de Entidades (Espelhamento de Dados)

O microserviço de Report possui as entidades `QuotationEntity` e `OpportunityEntity`.
Por que ele armazena a cotação no seu próprio banco de dados em vez de consultar diretamente a API ou o banco do microserviço de Cotação?

🟢 Atividade 5.3: Anotação de Escuta do Kafka

Para ler de um tópico no SmallRye Reactive Messaging, não usamos emissores manuais.
Qual anotação é utilizada acima de um método na classe `KafkaEvents` para assinar e ler continuamente as mensagens de um canal configurado?

🟢 Atividade 5.4: O Erro de Transação no Consumidor

Se você escutar o Kafka e tentar salvar no banco dentro do mesmo método, você pode tomar uma exceção `Transaction is not active`.
Como você resolve esse erro específico utilizando anotações na camada de escuta de eventos?

🟢 Atividade 5.5: Lógica de Cruzamento de Dados

Uma nova proposta chega via Kafka.
Como a classe `OpportunityServiceImpl` encontra a cotação correta (a mais recente) no banco de dados para criar a oportunidade de venda atualizada?

🟢 Atividade 5.6: A Camada de Utilitários (Utils)

No desenvolvimento, foi criado o pacote `utils` e a classe `CSVHelper`.
Por que lógicas como a geração e formatação de arquivos CSV não devem ficar diretamente dentro da classe `Controller` ou de `Entity`?

🟢 Atividade 5.7: Retornando Arquivos por API REST

O endpoint `/report` não retorna um JSON, mas sim um arquivo para download.
Qual anotação (`@Produces`) e *Media Type* é utilizado no `Controller` para indicar que a resposta será um fluxo de bytes (arquivo)?

🟢 Atividade 5.8: Múltiplas Inscrições de Kafka

O Report consome tanto `proposals` quanto `quotations`.
Descreva o que acontece no `application.properties` quando você precisa consumir dois canais diferentes. Quantas configurações de conector e tópicos são necessárias?

🟢 Atividade 5.9: Resiliência em Ação (Produtor Offline)

Se o microserviço de Cotação for desligado, o serviço de Report ainda assim deve continuar funcionando.
Explique por que o usuário final ainda consegue baixar os relatórios de propostas mesmo sem o sistema de cotação online.

🟢 Atividade 5.10: Recuperação de Mensagens Pendentes (Consumidor Offline)

Imagine que o microserviço de Report caia. Durante essa queda, 5 novas propostas são criadas.
O que acontece com essas 5 mensagens e qual será o comportamento do microserviço de Report assim que ele for reiniciado?
