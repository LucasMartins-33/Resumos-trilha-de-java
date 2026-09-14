# Questões - Capítulo 06: Segurança e Keycloak (Conceitos)

### Questão 1: O que é o Keycloak e qual sua utilidade na arquitetura?
<details>
<summary>👀 Ver Resposta</summary>

Keycloak é um servidor de gerenciamento de identidade e acesso (IAM) open source. Ele serve para centralizar a autenticação (Single Sign-On - SSO) e garantir que apenas usuários autorizados acessem os microsserviços.
</details>

### Questão 2: Em um ambiente empresarial, como grandes corporações gerenciam múltiplos sistemas sem precisar criar usuários diferentes para cada um?
<details>
<summary>👀 Ver Resposta</summary>

Utilizando o conceito de Single Sign-On (SSO) com diretórios centralizados como o LDAP ou Active Directory, muitas vezes mediados por servidores como o Keycloak, onde um único login (identidade federada) concede acesso a todas as ferramentas.
</details>

### Questão 3: Por que não se recomenda executar o Keycloak com o comando de inicialização `--start-dev` em produção?
<details>
<summary>👀 Ver Resposta</summary>

O modo `start-dev` usa configurações inseguras, chaves criptográficas em memória, modo de desenvolvimento habilitado e banco de dados H2 na memória, sendo voltado estritamente para testes rápidos e desenvolvimento local.
</details>

### Questão 4: O que é um "Realm" dentro do Keycloak?
<details>
<summary>👀 Ver Resposta</summary>

Um Realm é um domínio (ou "reino") isolado de segurança no Keycloak. Cada Realm gerencia seus próprios clientes, papéis (roles), grupos e usuários, de forma totalmente apartada de outros Realms.
</details>

### Questão 5: O que é o arquivo `realm.json` e para que ele serve neste projeto?
<details>
<summary>👀 Ver Resposta</summary>

É um arquivo exportado/preparado pela equipe do Quarkus contendo uma pré-configuração completa de um Realm (com clientes e roles prontas para teste). Ao importá-lo, o desenvolvedor poupa tempo na configuração inicial do Keycloak.
</details>

### Questão 6: Fazendo a analogia de segurança do instrutor, se o "login/senha" é a compra do ingresso, o que representa o Token JWT?
<details>
<summary>👀 Ver Resposta</summary>

O Token JWT representa a "pulseira do evento". Uma vez logado (ingresso validado na porta), o servidor te entrega a pulseira. Você passa a usá-la (Token JWT) para acessar as áreas restritas sem precisar mostrar a senha novamente.
</details>

### Questão 7: Quais são as três partes fundamentais que compõem um Token JWT?
<details>
<summary>👀 Ver Resposta</summary>

1. Header (Cabeçalho: tipo e algoritmo); 2. Payload (Corpo: dados do usuário, permissões, tempo de expiração); 3. Signature (Assinatura: chave criptográfica que garante que o token não foi adulterado).
</details>

### Questão 8: Se um invasor capturar um JWT e alterar o "Payload" para se transformar em um administrador, por que o servidor rejeitará a requisição?
<details>
<summary>👀 Ver Resposta</summary>

Porque ao alterar o Payload, a "Signature" (Assinatura) gerada pelo servidor original deixará de bater. O servidor possui a chave criptográfica para recalcular o JWT; ao perceber a divergência matemática, ele nega o acesso.
</details>

### Questão 9: O Token JWT gerado pelo Keycloak expira?
<details>
<summary>👀 Ver Resposta</summary>

Sim. É uma boa prática de segurança que o Payload do JWT contenha uma data de expiração (exp). Após esse tempo, o cliente precisará de um novo token (ou usar um *Refresh Token*) para continuar acessando as APIs.
</details>

### Questão 10: Dentro da organização do Keycloak, o que é um "Client"?
<details>
<summary>👀 Ver Resposta</summary>

É a entidade que solicita autenticação em nome de um usuário ou serviço. Na nossa arquitetura, os microsserviços (ex: `backend-service`) são os "Clients" que utilizam o Keycloak para validar a identidade e proteger os recursos.
</details>
