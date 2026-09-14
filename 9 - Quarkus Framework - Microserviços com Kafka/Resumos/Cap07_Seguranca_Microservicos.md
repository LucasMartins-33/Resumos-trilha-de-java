# Capítulo 07: Implementando Segurança nos Microserviços

Este documento resume o **Capítulo 07**, onde finalmente colocamos a mão no código para aplicar os conceitos de segurança do Keycloak nas nossas APIs REST, protegendo os endpoints para que não fiquem mais expostos publicamente na internet.

---

## 1. Escopo das Modificações de Segurança
Na nossa arquitetura atual:
- **Microserviço de Cotação:** Não sofre alterações. Como ele não expõe endpoints REST consumidos externamente (ele apenas busca dados de fora e joga no Kafka), o escopo dele já é fechado e seguro por natureza.
- **Microserviços de Proposta e Relatório (Report):** Serão blindados. Receberão dependências e configurações para validar Tokens JWT.

### Preparação para o API Gateway (BFF)
O instrutor explicou que, na arquitetura final, o usuário externo **nunca** chamará os serviços de Proposta e Report diretamente. Ele chamará um novo microserviço (API Gateway / BFF - Backend For Frontend) que será construído a seguir. O Gateway repassará a requisição para os serviços internos.

---

## 2. Dependências Adicionadas (`pom.xml`)
Foram adicionadas duas dependências essenciais nos projetos `proposta` e `report`:

1. **Quarkus OIDC (OpenID Connect):**
   ```xml
   <dependency>
       <groupId>io.quarkus</groupId>
       <artifactId>quarkus-oidc</artifactId>
   </dependency>
   ```
   *Função:* Habilita a proteção de endpoints, conecta-se ao Keycloak e valida os tokens JWT recebidos.

2. **Propagação de Token OIDC Reativo:**
   ```xml
   <dependency>
       <groupId>io.quarkus</groupId>
       <artifactId>quarkus-oidc-token-propagation-reactive</artifactId>
   </dependency>
   ```
   *Função:* **Crucial para a arquitetura.** Como a requisição passará pelo Gateway e depois irá para os microsserviços internos, essa biblioteca garante que o token JWT enviado pelo usuário seja "repassado" (propagado) nas requisições HTTP internas entre os microsserviços automaticamente. Sem ela, o Gateway seria barrado ao tentar acessar o Report.

---

## 3. Protegendo os Endpoints (Controladores)

Para proteger os endpoints, o instrutor injetou a interface `JsonWebToken` e utilizou as anotações de segurança do Java (JAX-RS / Jakarta Security):

### Microserviço de Proposta (`ProposalController`)
> ⚠️ **Atualização Quarkus 3:** As anotações de segurança como `@Authenticated` e `@RolesAllowed` agora devem ser importadas do pacote `jakarta.annotation.security.*` (e não mais de `javax.annotation.security.*`).

O instrutor demonstrou o controle de acesso baseado em papéis (Role-Based Access Control - RBAC):
- `@Authenticated` na classe: Garante que *qualquer* requisição precise de um token válido.
- `@RolesAllowed({"user", "manager"})` no `GET`: Apenas operadores de negócio ou gerentes podem buscar os detalhes de uma proposta. Clientes não.
- `@RolesAllowed("proposal-customer")` no `POST`: **Somente clientes** logados podem inserir novas propostas. Nenhum funcionário da mineradora consegue fraudar inserindo propostas em nome do cliente.
- `@RolesAllowed("manager")` no `DELETE`: Apenas o Gerente supremo tem poder de deletar uma proposta do banco.

### Microserviço de Report (`OpportunityController`)
- Adicionada a anotação `@Authenticated` e injetado o `JsonWebToken`.

---

## 4. Refatoração no Microserviço de Report (Delegação de Responsabilidade)
Houve uma mudança importante de arquitetura neste capítulo:
- **Exclusão do `CSVHelper`:** O instrutor **deletou** a classe que gerava o arquivo `.csv` de dentro do microserviço de Report.
- **Motivo (Padrão BFF):** A responsabilidade de decidir *como* os dados serão entregues ao usuário (seja em JSON para desenhar uma tela web, ou em formato de download CSV) é da camada de apresentação, ou seja, o **BFF (Backend For Frontend)**.
- **Nova lógica:** O microserviço de Report agora apenas vai no banco de dados e devolve a lista crua (`List<OpportunityDTO>`). O futuro Gateway/BFF pegará essa lista e, se o cliente pediu um CSV, ele mesmo se encarregará de montar e entregar o arquivo.

---

## 5. Configuração no `application.properties`
Para que os microsserviços parem de iniciar um Keycloak temporário (recurso do Dev Services do Quarkus) e apontem para o nosso Keycloak oficial (rodando no Docker na porta 8180), foi necessário adicionar as credenciais do `realm.json` criado na aula passada:

```properties
# Configurações de Segurança e conexão com o Keycloak
quarkus.oidc.auth-server-url=http://localhost:8180/realms/quarkus
quarkus.oidc.client-id=backend-service
quarkus.oidc.credentials.secret=secret
```

> **Resultado do Teste:** O instrutor abriu o Postman e tentou criar uma proposta sem enviar o Token. O servidor respondeu brilhantemente com o status **`401 Unauthorized`**. A barreira está funcionando! O próximo passo é construir o Gateway para orquestrar essas requisições.
