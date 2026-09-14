# Questões - Capítulo 09: Configurando Keycloak

### Questão 1: O modo de execução do Docker `start-dev` no Keycloak é ideal para ambientes de produção?
<details>
<summary>👀 Ver Resposta</summary>

Não. O modo `start-dev` usa chaves temporárias, banco H2 em memória, protocolo HTTP inseguro e expõe funções de desenvolvimento, devendo ser usado unicamente em ambientes de teste/local.
</details>

### Questão 2: Qual a principal facilidade de se usar o comando `Import` com o arquivo `quarkus-realm.json`?
<details>
<summary>👀 Ver Resposta</summary>

Evitar trabalho manual repetitivo. O arquivo já contém a arquitetura pronta de um *Realm* focado para a estrutura Quarkus, contendo *Clients* (como `backend-service`), além de *Roles* pré-definidas.
</details>

### Questão 3: Ao configurar os usuários locais de teste, por que o instrutor desligou a flag "Temporary" ao inserir as senhas?
<details>
<summary>👀 Ver Resposta</summary>

Para impedir que o Keycloak force a redefinição de senha logo no primeiro login. Desligar a flag garante que a senha configurada (ex: `1234`) fique persistente para testes fluídos.
</details>

### Questão 4: Onde são definidos, na interface do Keycloak, os papéis que um usuário específico pode assumir no sistema?
<details>
<summary>👀 Ver Resposta</summary>

Na aba "Role Mappings" presente dentro das configurações detalhadas de cada usuário criado.
</details>

### Questão 5: O que representa o "Client" chamado `backend-service` criado dentro do Realm?
<details>
<summary>👀 Ver Resposta</summary>

Representa as nossas aplicações Quarkus. Toda aplicação (ou grupo de microsserviços) precisa de um "Client ID" e uma "Secret" válidos cadastrados no Realm para poderem se autenticar perante o servidor de IAM.
</details>

### Questão 6: Por que foram criados usuários específicos como `China Miner` e outros como `João`?
<details>
<summary>👀 Ver Resposta</summary>

Para simular papéis diferentes na vida real. O "China Miner" atua com o papel de cliente (`proposal-customer`), enquanto o "João" é um funcionário (`user`), validando regras de negócio distintas.
</details>

### Questão 7: É possível, sem reiniciar os microsserviços, alterar as permissões de um usuário direto no painel do Keycloak?
<details>
<summary>👀 Ver Resposta</summary>

Sim. A vantagem de um sistema IAM centralizado como o Keycloak é que você revoga papéis ou bloqueia contas pelo painel, e as APIs do Quarkus bloquearão o usuário imediatamente após o término da validade do último token.
</details>

### Questão 8: Quando falamos em `client_secret` nas configurações OIDC do Quarkus, de onde retiramos esse valor?
<details>
<summary>👀 Ver Resposta</summary>

Da aba "Credentials" dentro das configurações do *Client* (no nosso caso, `backend-service`) no painel de administração do próprio Keycloak.
</details>

### Questão 9: Em vez de subir um container Docker, é possível configurar o Keycloak no código do Quarkus em desenvolvimento?
<details>
<summary>👀 Ver Resposta</summary>

Sim, o recurso de "Dev Services" do Quarkus inicia automaticamente um container do Keycloak caso detecte as dependências OIDC sem encontrar a configuração explícita, facilitando setups muito rápidos.
</details>

### Questão 10: Por que criar diferentes Roles (papéis) como `user` e `manager`?
<details>
<summary>👀 Ver Resposta</summary>

Para implementar o controle de acesso granular (RBAC). Ambos podem ver relatórios, mas a regra de negócio exige que exclusões de tabelas do banco só sejam permitidas aos "gerentes" (`manager`), e o Keycloak aplica essa divisão.
</details>
