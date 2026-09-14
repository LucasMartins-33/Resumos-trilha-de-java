📘 Capítulo 07: Implementando Segurança nos Microserviços

O Cenário:
O ecossistema Keycloak está explicado. Agora, você precisou modificar os códigos dos microsserviços de Proposta e de Report para validar os tokens JWT antes de processarem as requisições.

Sua missão é exercitar as mudanças em dependências, configurações e controle de papéis (RBAC):

🟢 Atividade 7.1: Dependência de Proteção

Qual dependência específica do Quarkus foi adicionada no `pom.xml` para habilitar a conexão com o Keycloak e validar tokens OIDC (OpenID Connect)?

🟢 Atividade 7.2: O Problema da Propagação de Token

O instrutor adicionou a dependência `quarkus-oidc-token-propagation-reactive`.
Se essa dependência não tivesse sido adicionada, o que aconteceria quando o Gateway (BFF) tentasse chamar o microserviço de Report repassando a requisição do usuário?

🟢 Atividade 7.3: Atualização de Pacotes do Java/Jakarta

No Quarkus 3+, de qual pacote (import) vêm as anotações de segurança como `@Authenticated` e `@RolesAllowed`, substituindo o antigo `javax.*`?

🟢 Atividade 7.4: Bloqueio Total

No Microserviço de Proposta (`ProposalController`), foi colocada a anotação `@Authenticated` no nível da classe.
Qual é o efeito prático de colocar essa anotação no topo da classe controladora em relação aos métodos HTTP que estão dentro dela?

🟢 Atividade 7.5: Controle Baseado em Papéis (RBAC)

O método HTTP POST que cria propostas recebeu a anotação `@RolesAllowed("proposal-customer")`.
Por que não seria seguro permitir que as permissões `user` ou `manager` (funcionários internos da empresa) também chamassem essa rota?

🟢 Atividade 7.6: Delegação de Exclusão

Apenas o gerente (manager) pode deletar propostas. 
Escreva como ficaria a assinatura e a anotação de segurança do método `removeProposal(long id)` no Controller para garantir isso.

🟢 Atividade 7.7: Mudança Arquitetural - O Fim do CSVHelper

O instrutor deletou a classe `CSVHelper` de dentro do microserviço de Report.
Baseado no conceito de "Backend For Frontend" (BFF), explique por que a responsabilidade de montar um arquivo CSV não deve ser do microserviço de domínio (Report).

🟢 Atividade 7.8: Recuperação da URL do Servidor de Autorização

No arquivo `application.properties`, como o Quarkus descobre em qual endereço de rede o Keycloak está rodando para validar se a assinatura do token é verdadeira? (Indique a propriedade de configuração).

🟢 Atividade 7.9: Configuração de Autenticação Interna

Ao apontar o `application.properties` para o Keycloak, também precisamos informar qual "client" o microserviço representa.
Quais propriedades são utilizadas para definir o ID do cliente (`client-id`) e a senha secreta (`credentials.secret`)?

🟢 Atividade 7.10: O Status HTTP de Rejeição

Após blindar as APIs, o instrutor tentou fazer uma requisição no Postman para a porta do microserviço de Proposta sem enviar o token.
Qual é o código de *Status HTTP* padronizado que o Quarkus retorna para avisar que a requisição não possui credenciais (ou que as credenciais são inválidas)?
