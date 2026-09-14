# Questões - Capítulo 07: Implementando Segurança (Quarkus)

### Questão 1: Quais dependências foram adicionadas no `pom.xml` para integrar o Quarkus com o Keycloak?
> [!success]- Resposta
> `quarkus-oidc` (para validar tokens com OpenID Connect) e `quarkus-oidc-token-propagation-reactive` (para propagar o token entre os microsserviços internos).

### Questão 2: Qual a principal função da biblioteca `quarkus-oidc-token-propagation-reactive` na nossa arquitetura?
> [!success]- Resposta
> Garantir que o Token JWT recebido no microserviço principal (Gateway) seja automaticamente anexado nos cabeçalhos (headers) ao fazer chamadas HTTP REST para os microsserviços internos, mantendo a autenticação ativa em toda a cadeia.

### Questão 3: Para que serve a anotação `@Authenticated` quando colocada acima de uma classe de Controller (Resource)?
> [!success]- Resposta
> Ela determina que todos os endpoints (métodos) daquela classe exigirão um Token JWT válido para serem acessados, independentemente das "Roles" do usuário.

### Questão 4: Como o Quarkus permite que apenas usuários "Gerentes" (Managers) deletem propostas?
> [!success]- Resposta
> Usando o RBAC (Role-Based Access Control) através da anotação `@RolesAllowed("manager")` diretamente em cima do método de `DELETE` no Controller.

### Questão 5: O que acontece se a anotação `@RolesAllowed` não encontrar um servidor OIDC configurado no `application.properties` em tempo de desenvolvimento no Quarkus?
> [!success]- Resposta
> O Quarkus (através do recurso *Dev Services*) tenta instanciar e subir automaticamente um container do Keycloak em background para não quebrar a aplicação durante os testes locais.

### Questão 6: Por que a classe `CSVHelper` foi excluída do microsserviço de Report neste capítulo?
> [!success]- Resposta
> Devido ao padrão BFF (Backend For Frontend). A responsabilidade de formatar dados para o cliente (gerar um CSV ou renderizar uma tela) é da camada de apresentação (Gateway). O microserviço de Report passou a retornar apenas JSON bruto.

### Questão 7: Em um cenário real, se não utilizarmos propagação de Token e chamarmos o Report a partir do Gateway, o que acontece?
> [!success]- Resposta
> O Gateway fará a chamada REST ao Report de forma "anônima". Como o Report está trancado, ele recusará a conexão com `401 Unauthorized`, quebrando a aplicação.

### Questão 8: Nas versões do Quarkus 3+, de qual pacote vêm as anotações `@RolesAllowed` e `@Authenticated`?
> [!success]- Resposta
> Elas pertencem ao pacote `jakarta.annotation.security.*` (substituindo o antigo `javax.annotation.security.*` das versões anteriores).

### Questão 9: Quais propriedades do `application.properties` são essenciais para conectar o Quarkus OIDC ao Keycloak rodando no Docker?
> [!success]- Resposta
> `quarkus.oidc.auth-server-url` (endereço do Keycloak e do Realm), `quarkus.oidc.client-id` (nome do cliente no Realm) e `quarkus.oidc.credentials.secret` (senha do client).

### Questão 10: O que retorna o servidor para o cliente caso o Token falte, esteja inválido ou tenha expirado?
> [!success]- Resposta
> O status code HTTP `401 Unauthorized` (Não Autorizado).
