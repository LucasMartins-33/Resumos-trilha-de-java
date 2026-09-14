📘 Capítulo 09: Configurando o Keycloak

O Cenário:
O Keycloak está em pé e você precisa configurá-lo. Apesar da facilidade de importar o Realm automatizado, é essencial entender o que está sendo criado por trás dos panos: papéis (roles), clientes (clients) e usuários.

Sua missão é testar a compreensão administrativa do painel do Keycloak:

🟢 Atividade 9.1: O Client "backend-service"

Durante a configuração do Realm "quarkus", o client `backend-service` já veio configurado.
Qual tipo de credencial esse Client fornece para que o arquivo `application.properties` da nossa aplicação Java consiga autenticar que pertence àquele Realm?

🟢 Atividade 9.2: Mapeamento de Roles vs Regras de Negócio

No Keycloak, criamos a Role `proposal-customer`.
Qual o efeito imediato na segurança da aplicação Quarkus (Gateway) se um novo usuário for criado no Keycloak mas o administrador esquecer de atribuir essa Role a ele?

🟢 Atividade 9.3: As Permissões do "manager"

De acordo com o mapeamento feito, o usuário "José" (`manager`) pode deletar relatórios e ver dados gerais.
No código, se o método `@GET` do Controller tiver `@RolesAllowed({"user", "manager"})`, o usuário "José" terá o acesso liberado? Por quê?

🟢 Atividade 9.4: Senha Temporária (Temporary Password)

Ao criar o usuário "João" com senha `1234`, o instrutor desligou a flag "Temporary" (OFF).
O que aconteceria no fluxo de login (via Postman) se a flag "Temporary" estivesse "ON"?

🟢 Atividade 9.5: A Diferença de Responsabilidades (Roles)

Houve a necessidade de criar a role `proposal-customer` exclusiva para clientes externos.
Baseado nas regras da "BR Mineradora", descreva um cenário catastrófico caso a role `user` (funcionário) tivesse a permissão `@RolesAllowed` adicionada no método de criação de propostas.

🟢 Atividade 9.6: Hierarquia de Roles (Roles Compostas)

O Keycloak permite criar "Roles Compostas", onde uma role "herda" as permissões de outra (Ex: um `manager` herda as permissões de `user`).
Se tivéssemos feito isso no painel do Keycloak, como o código do Quarkus precisaria ser alterado nas anotações `@RolesAllowed({"user", "manager"})` para métodos de leitura?

🟢 Atividade 9.7: Clientes OIDC (OpenID Connect)

O Keycloak pode atuar como um provedor OIDC (OpenID Connect).
O JWT é gerado nesse padrão. Sabendo disso, o Quarkus no lado do Gateway usa qual protocolo subjacente (usado pelo OIDC) para descobrir as chaves públicas e validar o token fornecido? (Dica: é a extensão configurada no `application.properties`).

🟢 Atividade 9.8: Criação de Múltiplos Realms

Poderíamos usar o Realm padrão "Master".
Cite uma boa razão prática (de segurança ou administração) para criarmos o nosso próprio Realm "quarkus" isolado para esta POC em vez de misturar com as configurações do Master.

🟢 Atividade 9.9: Mapeamento de Grupo vs Usuário Individual

Em vez de atribuir a role `user` usuário por usuário (João, Maria, Pedro).
Qual recurso o Keycloak oferece na interface para aplicar papéis (Roles) a um conjunto de pessoas de uma única vez, facilitando a vida do setor de TI?

🟢 Atividade 9.10: O Risco do Modo de Desenvolvimento

Revisando o início da aula: O instrutor avisa que em Produção não se deve usar senhas fáceis nem `start-dev`.
Na vida real de uma implantação, o painel do Keycloak deve ficar acessível para a Internet pública ou restrito a uma rede VPN de infraestrutura? Explique por quê.
