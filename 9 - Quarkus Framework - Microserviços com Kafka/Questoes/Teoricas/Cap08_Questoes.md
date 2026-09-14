# Questões - Capítulo 08: Microserviço Gateway (BFF)

### Questão 1: O que significa a sigla BFF e qual a sua função em arquiteturas de microsserviços?
> [!success]- Resposta
> BFF significa "Backend For Frontend". É um microserviço (ou camada Gateway) que serve como ponto único de entrada para o Frontend, roteando e adaptando as requisições para os microsserviços internos, aliviando-os da lógica de apresentação.

### Questão 2: Por que criar rotas no BFF em vez de deixar o Frontend chamar as APIs diretamente?
> [!success]- Resposta
> Isso reduz o acoplamento, evita que o frontend precise lidar com diferentes portas e endereços de dezenas de microsserviços, melhora a segurança e centraliza tratativas como CORS e formatação de respostas.

### Questão 3: O que faz um `RestClient` (via anotação `@RegisterRestClient`) no Quarkus?
> [!success]- Resposta
> Ele permite fazer requisições HTTP para outras APIs externas ou microsserviços internos, funcionando como um cliente HTTP embutido que se mapeia através de interfaces tipadas no Java.

### Questão 4: O microserviço de Gateway possui pacotes `repository` e se conecta diretamente a um banco de dados próprio?
> [!success]- Resposta
> Não. O Gateway não armazena dados de negócio; sua função é apenas atuar como intermediário. Ele consulta os dados através dos *REST Clients* batendo nas APIs do Report e do Proposal.

### Questão 5: Como o BFF implementa a funcionalidade de baixar um arquivo CSV?
> [!success]- Resposta
> O BFF chama a API do microserviço de Report que retorna os dados crus (JSON). Em seguida, dentro da sua camada *Service*, ele roda a classe `CSVHelper` formatando a lista de dados num `ByteArrayInputStream` e retornando ao usuário.

### Questão 6: Qual cabeçalho HTTP (`MediaType`) o Gateway retorna no seu *Controller* para avisar o navegador que a resposta é um download de arquivo?
> [!success]- Resposta
> O `MediaType.APPLICATION_OCTET_STREAM`.

### Questão 7: Para conectar o BFF (porta 8095) ao serviço de Proposta (porta 8091), onde configuramos essa URL?
> [!success]- Resposta
> No `application.properties`, apontando a URL base atrelada ao nome da interface REST Client. Ex: `quarkus.rest-client."...ProposalRestClient".url=http://localhost:8091`.

### Questão 8: No método que criar novas propostas, como o Gateway BFF gerencia as falhas que ocorrem no microserviço backend?
> [!success]- Resposta
> Ele lê o status HTTP retornado pelo serviço interno. Se for entre 200 e 204, ele devolve OK para o cliente final; caso fuja dessa faixa (ex: erros 4xx ou 5xx), ele captura e repassa esse status estranho para informar o erro.

### Questão 9: Em termos de segurança (OIDC), o Gateway precisa também validar os Tokens que chegam?
> [!success]- Resposta
> Sim, ele funciona como a "porta do condomínio". O Gateway valida o token no Keycloak usando o OIDC e, se válido, a biblioteca de *token propagation* injeta o token nos *REST Clients* para enviar aos microsserviços do fundo.

### Questão 10: Como o BFF lida com endpoints que necessitam entregar JSON vs CSV?
> [!success]- Resposta
> Ele cria endpoints separados (ex: `/data` e `/report`). O Controller aciona lógicas diferentes no Service, embora ambas puxem a informação do mesmíssimo endpoint backend, centralizando a lógica de apresentação.
