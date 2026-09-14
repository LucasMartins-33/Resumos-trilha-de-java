# Capítulo 08: Desenvolvendo o Microserviço Gateway (BFF)

Este documento contém o resumo do **Capítulo 08**, onde criamos o microserviço **Gateway** utilizando o padrão arquitetural **BFF (Backend For Frontend)**. Ele atuará como a porta de entrada centralizada da nossa aplicação, orquestrando e roteando as requisições externas para os microsserviços internos de Proposta e Relatório.

---

## 1. O Padrão BFF (Backend For Frontend)
O BFF serve para tirar a complexidade dos microsserviços de negócio (backend) e centralizar as lógicas de apresentação (para o frontend). 
- **O problema resolvido:** Se tivermos um app Android e um app Web precisando de formatos diferentes (ex: JSON vs CSV), não precisamos poluir a API de negócio. A API do microserviço de Report retorna apenas os dados puros, e o BFF formata a resposta conforme quem a requisitou.

---

## 2. Inicialização do Projeto e Dependências (`pom.xml`)
Foi criado o novo projeto Quarkus (`gateway-bff`) e adicionadas as seguintes dependências-chave:
1. **`quarkus-oidc`** e **`quarkus-oidc-token-propagation-reactive`**: Fundamental para proteger as rotas do Gateway e **repassar automaticamente** o Token JWT recebido nas requisições REST para os microsserviços internos.
2. **`smallrye-openapi`**: Adicionado para gerar documentação com Swagger (será configurado futuramente).
3. **`apache-commons-csv`**: Para gerar os relatórios CSV a partir do Gateway.

---

## 3. Estrutura de Pacotes
O projeto Gateway foi organizado da seguinte forma:
- **`utils`**: Onde colamos a classe `CSVHelper`. (A responsabilidade de gerar o CSV saiu do microserviço de Report e veio para cá).
- **`dto`**: Contém `OpportunityDTO` e `ProposalDetailsDTO` para transitar as informações recebidas dos outros microsserviços.
- **`client`**: Contém os *REST Clients* (interfaces que ensinam o Quarkus a fazer requisições HTTP para os microsserviços internos).
- **`service`**: Contém a lógica que intercepta as chamadas, consulta os *REST Clients* e trata as respostas.
- **`controller`**: As APIs REST expostas pelo Gateway para o mundo externo.

---

## 4. REST Clients e `application.properties`
Para que o Gateway consiga falar com os outros microsserviços, o instrutor criou duas interfaces: `ProposalRestClient` e `ReportRestClient`. As URLs de destino foram parametrizadas no `application.properties`:

```properties
# Porta do Gateway
quarkus.http.port=8095

# Rotas dos Rest Clients
quarkus.rest-client."org.br.mineradora.client.ProposalRestClient".url=http://localhost:8091
quarkus.rest-client."org.br.mineradora.client.ReportRestClient".url=http://localhost:8081

# Configurações do Keycloak e OIDC Client (para propagação do JWT)
quarkus.oidc.auth-server-url=http://localhost:8180/realms/quarkus
quarkus.oidc.client-id=backend-service
quarkus.oidc.credentials.secret=secret
```

---

## 5. Implementação da Camada de Serviços
Na camada Service, o instrutor injetou os REST Clients (usando a anotação correspondente) e criou a lógica central:
- **`ProposalServiceImpl`**: Basicamente serve de ponte (pass-through) para as ações de CRUD de propostas, delegando para o microserviço da porta 8091.
- **`ReportServiceImpl`**: Implementou os métodos `generateCSVOpportunityReport` e `getOpportunitiesData`. 
  - Ambos chamam a API do Report na porta 8081 para pegar a lista pura (`List<OpportunityDTO>`).
  - A lógica difere no retorno: O primeiro pega a lista, joga no `CSVHelper` e retorna um `ByteArrayInputStream` (arquivo). O segundo apenas retorna a lista em JSON.

---

## 6. Criação dos Controllers (Endpoints Externos)

Os controladores do Gateway são os endpoints que os clientes (Postman/Frontends) irão realmente chamar:

### `ProposalController` (Rota `/api/trade`)
Injeta o serviço e implementa regras severas de controle por papéis (`@RolesAllowed`):
- `GET` (Detalhes): Aberto para `{"user", "manager"}`.
- `POST` (Criar): Aberto estritamente para `"proposal-customer"`.
- `DELETE` (Remover): Acesso apenas para `"manager"`.
*Obs: O instrutor programou o Gateway para capturar os HTTP Status (ex: 200 a 204) retornados pelo microserviço interno e repassá-los ao usuário de forma transparente.*

### `ReportController` (Rota `/api/opportunity`)
Possui rotas separadas, ambas protegidas para `{"user", "manager"}`:
- `/report`: Chama o serviço que retorna `ByteArrayInputStream` e configura o *Media Type* como `APPLICATION_OCTET_STREAM` para forçar o download do `.csv`.
- `/data`: Devolve os mesmos dados, mas em uma resposta simples JSON.

> **Resumo Final:** Com o Gateway 100% desenvolvido, na próxima aula daremos vida ao **Keycloak**, simularemos o login real do usuário para pegar a "pulseira" (Token JWT) e faremos a chamada no Gateway para testar todo esse ecossistema orquestrado!
