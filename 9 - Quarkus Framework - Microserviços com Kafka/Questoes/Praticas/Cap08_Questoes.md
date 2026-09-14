📘 Capítulo 08: Desenvolvendo o Microserviço Gateway (BFF)

O Cenário:
Para não expor os microsserviços de negócio (Proposta, Report) diretamente para a internet e para centralizar a criação de arquivos de apresentação (CSV), foi introduzido o Gateway com o padrão BFF (Backend For Frontend).

Sua missão é exercitar a criação de orquestração de chamadas HTTP, REST Clients e o roteamento de serviços:

🟢 Atividade 8.1: O Padrão Backend For Frontend (BFF)

Dê um exemplo prático de um cenário em que a existência de um BFF é crucial quando temos múltiplos tipos de clientes (como um App Mobile e um Painel Web) que consomem a mesma regra de negócio.

🟢 Atividade 8.2: REST Clients no Quarkus

O Gateway não acessa bancos de dados, ele chama as APIs dos outros microsserviços.
Como o Quarkus permite que criemos uma interface Java (ex: `ProposalRestClient`) que saiba fazer requisições HTTP para a porta 8091 sem precisarmos escrever a lógica complexa do HTTP Client?

🟢 Atividade 8.3: A Ponte (Pass-Through) de Propostas

No `ProposalServiceImpl` do Gateway, os métodos apenas chamam o REST Client do serviço interno.
Se o microserviço interno de Proposta na porta 8091 retornar um erro 400 Bad Request, o que o Gateway faz com esse erro em relação ao usuário final?

🟢 Atividade 8.4: Formatação de Respostas (Report)

O serviço `ReportServiceImpl` agora orquestra a geração de dados e CSV.
Se o front-end solicitar os dados chamando o método `/api/opportunity/data`, qual será o formato retornado? E se chamar `/api/opportunity/report`?

🟢 Atividade 8.5: Trabalhando com Arquivos Binários

Para baixar o CSV através do Gateway, o Controller manipula um `ByteArrayInputStream`.
Qual anotação do JAX-RS (Jakarta) define que o tipo de retorno (`MediaType`) não será JSON, mas sim um formato binário de download?

🟢 Atividade 8.6: Configuração Dinâmica de URLs

No `application.properties` do Gateway, precisamos mapear as interfaces do REST Client para a URL real do microserviço destino.
Como fica a chave de configuração para associar a URL `http://localhost:8081` à interface `ReportRestClient`?

🟢 Atividade 8.7: Rotas Externas (Controllers do Gateway)

O Gateway expõe a rota `/api/trade` para as propostas e `/api/opportunity` para relatórios.
Por que é uma boa prática criar esses "alias" ou rotas consolidadas no Gateway em vez de usar exatamente as mesmas rotas internas?

🟢 Atividade 8.8: Repassando o Token (Token Propagation)

Como garantimos que o JWT que o usuário enviou ao Gateway chegue intacto ao microserviço interno ao utilizar o REST Client? Qual dependência reativa faz isso de forma invisível?

🟢 Atividade 8.9: Isolamento de Rede (Segurança)

Do ponto de vista da infraestrutura de nuvem, se os microsserviços internos estão em uma sub-rede privada e não possuem IP público.
Qual é a única máquina ou serviço que deve receber um IP público para que a arquitetura funcione perfeitamente via Postman?

🟢 Atividade 8.10: Dependência de Documentação

No final do POM.xml do Gateway, o instrutor adicionou o `smallrye-openapi`.
Qual é o objetivo principal de adicionar essa dependência no serviço de borda (Gateway)?
