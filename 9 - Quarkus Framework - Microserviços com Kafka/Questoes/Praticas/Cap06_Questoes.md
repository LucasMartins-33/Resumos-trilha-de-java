📘 Capítulo 06: Pensando na Segurança da Aplicação (Keycloak e JWT)

O Cenário:
Com os microsserviços funcionando, percebemos que qualquer pessoa poderia fazer requisições e alterar nosso banco de dados. Para resolver isso, introduziremos o Keycloak e a tecnologia de JWT.

Sua missão é testar a compreensão dos conceitos básicos de Identity and Access Management e da estrutura do Token JWT:

🟢 Atividade 6.1: O que é SSO (Single Sign-On)?

Explique como o Keycloak facilita a vida de um funcionário que precisa utilizar 5 sistemas diferentes dentro da mesma empresa utilizando a técnica de SSO.

🟢 Atividade 6.2: Modo de Execução do Keycloak

O instrutor subiu o Keycloak via Docker utilizando a flag `start-dev`.
Qual é o grande perigo de utilizar essa flag de inicialização e credenciais fáceis (admin/admin) em um ambiente de produção?

🟢 Atividade 6.3: O que é um Realm?

Dentro da arquitetura do Keycloak, as configurações não ficam soltas.
Defina com suas palavras o que é um "Realm" e por que nós não utilizamos o Realm "Master" para cadastrar nossos clientes e usuários da BR Mineradora.

🟢 Atividade 6.4: O que é um Client no Keycloak?

No arquivo `realm.json` importado, existe um client chamado `backend-service`.
Qual é a relação entre esse "Client" configurado no Keycloak e os microsserviços que programamos em Java?

🟢 Atividade 6.5: A Analogia do JWT

O instrutor fez uma comparação entre autenticação/autorização e uma "pulseira de evento".
Na prática de software, qual processo representa a entrega do "ingresso" (login) e qual processo representa a verificação da "pulseira" (validação do JWT)?

🟢 Atividade 6.6: Estrutura do JWT - O Header

Um token JWT é composto por 3 partes. A primeira é o Header.
Que tipo de informação vital para a segurança fica armazenada na seção de cabeçalho do token?

🟢 Atividade 6.7: Estrutura do JWT - O Payload

A segunda parte do JWT é o Payload.
Liste três informações comuns (claims) sobre o usuário que podem ser encontradas dentro do payload de um token gerado pelo Keycloak.

🟢 Atividade 6.8: Estrutura do JWT - A Assinatura (Signature)

A terceira parte é a Assinatura.
Explique como a assinatura impede que um hacker (ou usuário mal-intencionado) intercepte o token, altere seu papel de `user` para `admin` no payload, e tente reenviá-lo ao servidor.

🟢 Atividade 6.9: Importação Automatizada

Para o ambiente de desenvolvimento, o instrutor não clicou tela a tela para configurar o Realm.
Como é possível fazer o backup ou subir um ecossistema pronto de usuários e clientes de forma automatizada no Keycloak?

🟢 Atividade 6.10: Stateful vs Stateless

Com o uso do JWT, a aplicação Quarkus se torna *Stateless* (sem estado), ou seja, ela não precisa guardar a sessão do usuário na memória RAM (como ocorria no antigo Java EE).
Por que o formato JWT permite que a aplicação não precise guardar a sessão na memória?
