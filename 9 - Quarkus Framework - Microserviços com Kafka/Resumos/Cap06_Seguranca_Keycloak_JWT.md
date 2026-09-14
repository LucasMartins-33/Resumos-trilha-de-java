# Capítulo 06: Pensando na Segurança da Aplicação (Keycloak e JWT)

Este documento contém o resumo do **Capítulo 06**, que dá uma pausa no desenvolvimento de código para introduzir os conceitos teóricos e a configuração inicial da infraestrutura de segurança usando **Keycloak** e **Tokens JWT**. O objetivo é proteger as APIs REST dos microsserviços, que atualmente estão totalmente abertas para a internet.

---

## 1. O que é o Keycloak?
O Keycloak é um servidor de autenticação e autorização *Open Source* (Identity and Access Management - IAM). 

### Casos de Uso Comuns
- **Grandes Corporações (SSO / LDAP):** Uma empresa gigante possui dezenas de softwares (RH, Estoque, Finanças). Em vez de o funcionário criar um usuário e senha diferente para cada software, usa-se o conceito de **Single Sign-On (SSO)** conectado a uma base centralizada (LDAP ou Active Directory). O Keycloak atua no meio, garantindo que o funcionário use *o mesmo login* para todos os sistemas.
- **Login Social:** Implementar facilmente "Entrar com Google" ou "Entrar com Facebook".

### Objetivo no Projeto
Garantir que endpoints como `GET /api/opportunity/report` só possam ser consumidos por usuários que estejam devidamente autenticados e que possuam a "Role" (permissão) adequada.

---

## 2. Subindo o Keycloak via Docker (Ambiente de Desenvolvimento)
Em vez de baixar e instalar manualmente, o instrutor utiliza o **Docker** para subir o servidor Keycloak de forma isolada.

### Execução Básica
Foi executado um container do Keycloak em **Modo de Desenvolvimento** (`start-dev`), definindo as credenciais temporárias de administrador:
- **Usuário:** `admin`
- **Senha:** `admin`
- **Porta:** Mapeado para `8180` (Acesso via `http://localhost:8180`)

⚠️ *Nota do instrutor:* **Jamais** utilize o usuário `admin` com a senha `admin` e o modo `start-dev` em um ambiente de produção real!

---

## 3. Configuração do Realm (`realm.json`)
No Keycloak, um **Realm** representa um "reino" ou um domínio isolado de segurança (onde ficam guardados os clientes, usuários e papéis).

### O Papel do Desenvolvedor vs Segurança
Em empresas reais, uma equipe de infraestrutura/segurança da informação gerencia o Keycloak e cria o Realm. O desenvolvedor apenas programa a API para validar a segurança que a equipe criou.
Como estamos em desenvolvimento e testes locais, precisamos configurar nosso próprio Keycloak.

### Importação do Arquivo Pronto do Quarkus
Para não precisarmos configurar dezenas de propriedades complexas, a própria equipe do Quarkus disponibiliza um arquivo chamado `realm.json` pré-configurado para testes.
- **Como foi feito:** No console do Keycloak, o instrutor fez o **Import** deste `realm.json`.
- **O que ele criou:**
  - Um Realm chamado `quarkus`.
  - Um client chamado `backend-service` (que será o ID da nossa aplicação).
  - Papéis (Roles) pré-definidos: `user`, `admin`.
  - Usuários prontos para teste: `admin`, `alice`, `jdoe` (com a senha padrão `alice` para a usuária alice, por exemplo).

---

## 4. Como funciona o JWT (JSON Web Token)
O instrutor usou uma excelente analogia para explicar a autenticação moderna via *Tokens JWT*.

### A Analogia da Pulseira de Eventos
1. **O Ingresso (Login):** Você compra um ingresso para um show (usuário e senha).
2. **A Portaria (Autenticação):** Na porta, o segurança checa se o ingresso é verdadeiro. Se for, ele te dá uma **pulseira** (JWT).
3. **Livre Trânsito (Autorização):** Com a pulseira no braço, você pode sair para ir ao carro e voltar para o show. O segurança na porta não pede mais o seu ingresso, ele apenas **olha a sua pulseira**.
4. **Validade (Expiração):** Se o show acabar, a pulseira perde a validade.

### Estrutura do JWT em Softwares
O usuário envia Login e Senha para o Keycloak uma única vez. Se estiver tudo certo, o Keycloak devolve um "passaporte" (O Token JWT) gigante. O usuário então envia esse token nas próximas requisições. O Token JWT possui 3 partes:
1. **Header (Cabeçalho):** Diz qual algoritmo de criptografia foi usado.
2. **Payload (Dados):** Contém os dados do usuário. É aqui que diz se ele é "admin" ou "user", qual o e-mail dele, e qual o **tempo de expiração** do token.
3. **Signature (Assinatura):** É a segurança real. Uma assinatura criptografada que garante ao servidor que foi o próprio Keycloak quem gerou aquele token, e que nenhum hacker alterou o conteúdo (payload) no meio do caminho.

> **Próximos Passos:** No próximo capítulo, iremos voltar ao código (Microsserviços e Gateway) e programá-los para exigir essa "pulseira" (Token JWT) emitido pelo Keycloak!
