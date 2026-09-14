# Capítulo 09: Configurando o Keycloak

Este documento contém o resumo do **Capítulo 09**, onde subimos e configuramos a instância do Keycloak que atuará como nosso servidor de identidade (IAM). Criamos o nosso domínio de segurança (Realm), perfis (Roles) e os usuários para realizar os testes do nosso ecossistema de microsserviços.

---

## 1. Subindo o Servidor Keycloak via Docker
O instrutor utilizou o Docker para iniciar rapidamente uma instância de desenvolvimento do Keycloak:
- O container foi iniciado no modo `start-dev` mapeando a porta para a `8180` (Acesso via `http://localhost:8180`).
- Credenciais temporárias para acessar o console de administração:
  - **Usuário:** `admin`
  - **Senha:** `admin`
- ⚠️ *Alerta importante do instrutor: O modo `start-dev` e senhas fáceis jamais devem ser usados em ambiente de produção.*

---

## 2. Importação do Realm "Quarkus"
No Keycloak, a área isolada onde residem nossas configurações de usuários e clientes chama-se **Realm**.
Para poupar o trabalho de configurar um ecossistema inteiro do zero, foi utilizado o arquivo base disponibilizado pela própria equipe do Quarkus (`quarkus-realm.json`):
1. No menu superior esquerdo (onde fica escrito *Master*), clicou-se em **Add Realm**.
2. Foi feito o upload do arquivo `quarkus-realm.json`.
3. Isso criou um novo Realm chamado **quarkus**.
4. Esse Realm importado já traz configurado o *client* **`backend-service`** (com a senha secreta padrão `secret`), que é exatamente o que as propriedades (`application.properties`) do nosso Gateway e dos nossos microsserviços estão esperando para conseguir se conectar ao Keycloak.

---

## 3. Criação das Permissões (Roles)
Para refletir as regras de negócio de quem pode acessar o quê na BR Mineradora (conforme programado nos arquivos de Controller das aulas anteriores), as seguintes **Roles** foram configuradas na aba de Roles do Keycloak:
- **`user`**: O operador comum da mineradora. Pode buscar relatórios e ver os detalhes de propostas.
- **`manager`**: O gerente. Pode fazer tudo o que o `user` faz, com a diferença de ser o único capaz de apagar (deletar) uma proposta do banco de dados.
- **`proposal-customer`**: O cliente (empresas que compram minério). É o único perfil autorizado a criar/enviar novas propostas.

---

## 4. Cadastro dos Usuários de Teste
Na tela de *Users*, o instrutor criou os seguintes perfis, garantindo que em *Credentials* a senha fosse padronizada como `1234` e a flag *Temporary* estivesse em **OFF** (para que o Keycloak não obrigue a trocar a senha no primeiro acesso durante os testes locais):

| Usuário (Username) | Nome Completo | Role Atribuída (Role Mappings) | Perfil Prático |
| :--- | :--- | :--- | :--- |
| **João** | João Silva | `user` | Operador da BR Mineradora |
| **José** | José Silva | `manager` | Gerente da BR Mineradora |
| **Usinas America** | Usinas América | `proposal-customer` | Cliente comprador de minério |
| **China Miner** | China Miner | `proposal-customer` | Cliente comprador de minério |

> **Próximos Passos:** Com a infraestrutura pronta, nossos *Controllers* seguros, o *API Gateway* escutando as requisições e nosso banco de usuários cadastrado, o próximo capítulo focará em finalmente subir todos os serviços e utilizar o Postman para disparar as requisições simulando cada um desses usuários.
