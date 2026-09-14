# Questões - Capítulo 05: Microsserviço de Relatório

### Questão 1: Como o microsserviço de Report recebe as informações de cotação e propostas?
> [!success]- Resposta
> Através do Apache Kafka. Ele atua como um *Consumer*, escutando (assumindo a leitura) os tópicos onde os microsserviços de Cotação e Proposta publicaram seus eventos.

### Questão 2: Qual anotação do SmallRye Reactive Messaging é utilizada para ler (consumir) mensagens de um tópico do Kafka?
> [!success]- Resposta
> A anotação `@Incoming("nome-do-canal")`, colocada no método que processará a mensagem recebida.

### Questão 3: O que o instrutor demonstrou ao desligar propositalmente o microsserviço de Report e enviar propostas?
> [!success]- Resposta
> Ele demonstrou o conceito de resiliência e desacoplamento do Kafka. Com o Report offline, as mensagens ficaram retidas no Kafka. Quando o Report foi religado, ele leu imediatamente as mensagens acumuladas, sem perda de dados.

### Questão 4: Por que o DTO lido pelo consumidor no serviço de Report deve ter exatamente os mesmos campos do DTO enviado pelo produtor?
> [!success]- Resposta
> Para que a desserialização do JSON que trafega no Kafka ocorra perfeitamente. Se os campos, nomes ou tipos divergirem sem configuração prévia, a biblioteca Jackson falhará ao converter a mensagem JSON para o objeto Java.

### Questão 5: O que acontece se o microsserviço de Report tentar atualizar um registro no banco sem a anotação `@Transactional` no consumidor do Kafka?
> [!success]- Resposta
> A aplicação lançará uma exceção, pois o método acionado pela mensagem assíncrona do Kafka estará tentando escrever no banco de dados fora do escopo de uma transação ativa.

### Questão 6: A biblioteca `apache-commons-csv` foi inicialmente adicionada ao microsserviço de Report. Para que ela servia?
> [!success]- Resposta
> Para formatar a lista de dados (`OpportunityDTO`) recuperada do banco de dados num arquivo de formato CSV legível para o usuário final. (Posteriormente essa responsabilidade foi movida para o Gateway).

### Questão 7: Na comunicação síncrona vs assíncrona, por que a opção de usar chamadas REST foi descartada para alimentar o banco do Report?
> [!success]- Resposta
> Se usássemos REST, o microsserviço de Proposta ficaria acoplado e dependente do Report. Se o Report caísse, o cliente receberia um erro (ou ocorreria timeout) ao tentar criar uma proposta, arruinando a experiência do usuário.

### Questão 8: Qual é a responsabilidade do `OpportunityService` dentro do microserviço de Report?
> [!success]- Resposta
> Ele guarda a lógica de cruzar os dados. Quando um evento chega, ele lê as informações da cotação salva e cruza com as propostas que vão chegando para gerar e persistir "Oportunidades".

### Questão 9: Em vez de injetar dependências com o `@Autowired` (padrão Spring), qual anotação padrão do ecossistema CDI/Quarkus utilizamos para injetar os *Repositories* e *Services*?
> [!success]- Resposta
> A anotação `@Inject`.

### Questão 10: O microserviço de Report expõe uma API REST (Controller) inicialmente. Qual era a finalidade original desse Controller antes da criação do Gateway?
> [!success]- Resposta
> Atender requisições diretas de clientes/frontend, indo até o `OpportunityService` buscar os dados consolidados e devolvendo esses dados ou um arquivo CSV.
