# Questões - Capítulo 07: Implementando Segurança (Quarkus)

### Questão 1: Quais dependências foram adicionadas no `pom.xml` para integrar o Quarkus com o Keycloak?
<details>
<summary>👀 Ver Resposta</summary>

`quarkus-oidc` (para validar tokens com OpenID Connect) e `quarkus-oidc-token-propagation-reactive` (para propagar o token entre os microsserviços internos).
</details>

### Questão 2: Qual a principal função da biblioteca `quarkus-oidc-token-propagation-reactive` na nossa arquitetura?
<details>
<summary>👀 Ver Resposta</summary>

Garantir que o Token JWT recebido no microserviço principal (Gateway) seja automaticamente anexado nos cabeçalhos (headers) ao fazer chamadas HTTP REST para os microsserviços internos, mantendo a autenticação ativa em toda a cadeia.
</details>

### Questão 3: Para que serve a anotação `@Authenticated` quando colocada acima de uma classe de Controller (Resource)?
<details>
<summary>👀 Ver Resposta</summary>

Ela determina que todos os endpoints (métodos) daquela classe exigirão um Token JWT válido para serem acessados, independentemente das "Roles" do usuário.
</details>

### Questão 4: Como o Quarkus permite que apenas usuários "Gerentes" (Managers) deletem propostas?
<details>
<summary>👀 Ver Resposta</summary>

Usando o RBAC (Role-Based Access Control) através da anotação `@RolesAllowed("manager")` diretamente em cima do método de `DELETE` no Controller.
</details>

### Questão 5: O que acontece se a anotação `@RolesAllowed` não encontrar um servidor OIDC configurado no `application.properties` em tempo de desenvolvimento no Quarkus?
<details>
<summary>👀 Ver Resposta</summary>

O Quarkus (através do recurso *Dev Services*) tenta instanciar e subir automaticamente um container do Keycloak em background para não quebrar a aplicação durante os testes locais.
</details>

### Questão 6: Por que a classe `CSVHelper` foi excluída do microsserviço de Report neste capítulo?
<details>
<summary>👀 Ver Resposta</summary>

Devido ao padrão BFF (Backend For Frontend). A responsabilidade de formatar dados para o cliente (gerar um CSV ou renderizar uma tela) é da camada de apresentação (Gateway). O microserviço de Report passou a retornar apenas JSON bruto.
</details>

### Questão 7: Em um cenário real, se não utilizarmos propagação de Token e chamarmos o Report a partir do Gateway, o que acontece?
<details>
<summary>👀 Ver Resposta</summary>

O Gateway fará a chamada REST ao Report de forma "anônima". Como o Report está trancado, ele recusará a conexão com `401 Unauthorized`, quebrando a aplicação.
</details>

### Questão 8: Nas versões do Quarkus 3+, de qual pacote vêm as anotações `@RolesAllowed` e `@Authenticated`?
<details>
<summary>👀 Ver Resposta</summary>

Elas pertencem ao pacote `jakarta.annotation.security.*` (substituindo o antigo `javax.annotation.security.*` das versões anteriores).
</details>

### Questão 9: Quais propriedades do `application.properties` são essenciais para conectar o Quarkus OIDC ao Keycloak rodando no Docker?
<details>
<summary>👀 Ver Resposta</summary>

`quarkus.oidc.auth-server-url` (endereço do Keycloak e do Realm), `quarkus.oidc.client-id` (nome do cliente no Realm) e `quarkus.oidc.credentials.secret` (senha do client).
</details>

### Questão 10: O que retorna o servidor para o cliente caso o Token falte, esteja inválido ou tenha expirado?
<details>
<summary>👀 Ver Resposta</summary>

O status code HTTP `401 Unauthorized` (Não Autorizado).
</details>
