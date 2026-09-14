# Questões Práticas - Capítulo 14 (Testes de Integração com Testcontainers)

---

### 🟢 Nível 1: Adicionando as Dependências no `pom.xml`
**Cenário:** Você precisa preparar uma aplicação Spring Boot 3+ para executar testes de integração contra um banco de dados MySQL real rodando em contêiner Docker.
**Sua Tarefa:**
* No arquivo `pom.xml`, adicione as três dependências fundamentais:
  * `spring-boot-testcontainers` (escopo test).
  * `org.testcontainers:junit-jupiter` (escopo test).
  * `org.testcontainers:mysql` (escopo test).
* Explique a função de cada uma dessas bibliotecas na suíte de testes.

---

### 🟡 Nível 2: Declarando o Contêiner com a Anotação `@Container`
**Cenário:** Você quer que o JUnit 5 gerencie o ciclo de vida de um contêiner MySQL em uma classe de testes de integração.
**Sua Tarefa:**
* Crie a classe de teste `UsuarioIntegrationTest`.
* Anote a classe com `@Testcontainers` e `@SpringBootTest(webEnvironment = RANDOM_PORT)`.
* Declare o campo estático do contêiner com versão cravada:
  ```java
  @Container
  private static MySQLContainer<?> mysqlContainer = new MySQLContainer<>("mysql:8.4.0");
  ```
* Escreva um `@Test` validando `assertTrue(mysqlContainer.isRunning())`.

---

### 🟠 Nível 3: Conexão Automática com a Mágica do `@ServiceConnection`
**Cenário:** Antigamente era necessário configurar `@DynamicPropertySource` com métodos verbosos para injetar a porta do contêiner nas propriedades do Spring.
**Sua Tarefa:**
* Adicione a anotação `@ServiceConnection` no campo do contêiner:
  ```java
  @Container
  @ServiceConnection
  private static MySQLContainer<?> mysqlContainer = new MySQLContainer<>("mysql:8.4.0");
  ```
* Injete o repositório `@Autowired private UsersRepository usersRepository;`.
* Salve uma entidade e comprove que o Spring Boot conectou ao MySQL do contêiner sem precisar de nenhuma linha de configuração de URL no `application.properties`.

---

### 🔴 Nível 4: A Boa Prática da Versão Imutável vs `latest`
**Cenário:** Um desenvolvedor declarou o contêiner como `new MySQLContainer<>("mysql:latest")`.
**Sua Tarefa:**
* Explique por que utilizar a tag `latest` em suítes de testes automatizados é uma má prática em ambientes corporativos e esteiras de CI/CD.
* O que acontece se uma atualização automática da imagem Docker alterar comandos SQL ou comportamentos padrão do banco?
* Refatore a declaração para fixar uma versão semântica precisa (ex: `"mysql:8.4.0"`).

---

### 🟣 Nível 5: Inspecionando o Mapeamento Dinâmico de Portas
**Cenário:** O MySQL escuta internamente na porta padrão `3306`, mas você já tem uma instância local de MySQL rodando no seu computador na mesma porta.
**Sua Tarefa:**
* No método de teste, imprima a porta mapeada do host:
  `System.out.println("Porta dinâmica sorteada pelo Docker: " + mysqlContainer.getFirstMappedPort());`.
* Verifique se a porta sorteada é um número alto livre (ex: `49152`).
* Explique por que esse mapeamento randômico elimina conflitos com serviços locais.

---

### 🟤 Nível 6: Diagnosticando a Condição de Corrida (Race Condition)
**Cenário:** Em uma suíte estatal com `@TestInstance(PER_CLASS)` e `@TestMethodOrder`, o Spring Boot falha na inicialização com o erro: *"Mapped port can only be obtained when the container is running"*.
**Sua Tarefa:**
* Explique por que essa condição de corrida ocorre entre o ciclo de inicialização do Spring Boot e o boot do contêiner Docker pelo JUnit.
* Qual componente tenta resolver as propriedades antes do término do startup do contêiner?

---

### 🔵 Nível 7: Resolvendo a Sincronização com o Bloco Estático (`static { }`)
**Cenário:** Para solucionar a falha de sincronização do Nível 6, você precisa garantir que o contêiner termine seu boot antes que qualquer classe ou anotação do Spring seja carregada.
**Sua Tarefa:**
* Remova a anotação `@Container` do atributo do contêiner (mantendo `@ServiceConnection`).
* Inicialize o contêiner programaticamente utilizando um bloco estático do Java:
  ```java
  @ServiceConnection
  private static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.4.0");

  static {
      mysql.start();
  }
  ```
* Reexecute a suíte ordenada com `@TestInstance(PER_CLASS)` e comprove a inicialização perfeita sem falhas de porta.

---

### 🟢 Nível 8: Entendendo o Papel do Contêiner Auxiliar `Ryuk`
**Cenário:** Durante a execução do teste, você executa o comando `docker ps` no terminal e nota um contêiner chamado `testcontainers/ryuk` rodando simultaneamente.
**Sua Tarefa:**
* O que é o contêiner `Ryuk` e qual é a sua missão no ecossistema do Testcontainers?
* O que o `Ryuk` faz caso você interrompa a execução do teste abruptamente pelo botão "Stop" da IDE?
* Por que ele evita vazamento de recursos e acúmulo de contêineres órfãos na máquina do desenvolvedor?

---

### 🟡 Nível 9: Flexibilidade Multisserviço (PostgreSQL, Kafka, Redis)
**Cenário:** A arquitetura do sistema utiliza mensageria assíncrona com Apache Kafka e banco de dados PostgreSQL.
**Sua Tarefa:**
* Demonstre como declarar contêineres para outras tecnologias utilizando o mesmo padrão do Testcontainers:
  * Como declarar um contêiner de PostgreSQL (`PostgreSQLContainer<?>`).
  * Como declarar um contêiner de Apache Kafka (`KafkaContainer`).
* Explique por que o Testcontainers é considerado uma solução universal para dependências de infraestrutura em testes de integração.

---

### 🟠 Nível 10: Integração Final (Suíte E2E Completa com Banco Docker Real)
**Cenário:** Você precisa entregar a validação final de ponta a ponta da sua API bancária rodando sobre um contêiner MySQL real com Docker.
**Sua Tarefa:**
* Construa a classe `ContaBancariaDockerE2ETest`:
  * Inicialize o MySQL 8.4 via Testcontainers com `@ServiceConnection`.
  * Use `@SpringBootTest(webEnvironment = RANDOM_PORT)` com `TestRestTemplate`.
  * Execute o fluxo real:
    1. POST `/contas` para criar uma nova conta e gravar fisicamente nas tabelas do MySQL Docker.
    2. GET `/contas/{id}` para buscar o registro diretamente do banco real.
    3. PUT `/contas/{id}/deposito` para atualizar saldo com transação ACID real.
  * Valide se o comportamento do MySQL (como auto-incremento de ID e locks transacionais) opera exatamente como em produção.
