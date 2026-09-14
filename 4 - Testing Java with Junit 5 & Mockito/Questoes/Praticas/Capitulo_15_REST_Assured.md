# Questões Práticas - Capítulo 15 (Testando APIs RESTful com REST Assured)

---

### 🟢 Nível 1: Configuração Global com `@LocalServerPort` e `@BeforeEach`
**Cenário:** Você precisa preparar a classe de teste para apontar automaticamente o REST Assured para a porta dinâmica onde o Spring Boot inicializou o servidor.
**Sua Tarefa:**
* Crie a classe de teste `UsuariosRestAssuredTest`.
* Anote com `@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)`.
* Injete a porta: `@LocalServerPort private int port;`.
* Crie um método `@BeforeEach` configurando as variáveis estáticas globais:
  ```java
  RestAssured.baseURI = "http://localhost";
  RestAssured.port = this.port;
  ```

---

### 🟡 Nível 2: O Primeiro Teste Fluente BDD (Given, When, Then)
**Cenário:** Você quer disparar uma requisição GET simples para a rota `/users` e validar o status de resposta.
**Sua Tarefa:**
* Importe estaticamente os métodos da classe `io.restassured.RestAssured.*`.
* Escreva um método `@Test` com a sintaxe encadeada BDD:
  ```java
  given()
  .when()
      .get("/users")
  .then()
      .statusCode(200);
  ```
* Execute o teste e observe o resultado na IDE.

---

### 🟠 Nível 3: Enviando Parâmetros de Consulta (`queryParam`)
**Cenário:** O endpoint `/users` suporta paginação e filtros via query parameters (ex: `/users?page=1&limit=50`).
**Sua Tarefa:**
* Na seção `given()`, adicione os parâmetros de consulta:
  ```java
  given()
      .queryParam("page", 1)
      .queryParam("limit", 50)
  .when()
      .get("/users")
  .then()
      .statusCode(200);
  ```
* Verifique se os parâmetros são anexados à URL e processados corretamente pelo controller.

---

### 🔴 Nível 4: Enviando Parâmetros de Rota (`pathParam`)
**Cenário:** Você precisa buscar um recurso por ID dinâmico na rota `/users/{userId}`.
**Sua Tarefa:**
* Configure o parâmetro de caminho na requisição:
  ```java
  given()
      .pathParam("userId", "USER-12345")
  .when()
      .get("/users/{userId}")
  .then()
      .statusCode(200);
  ```
* Explique a diferença conceitual entre `pathParam` e `queryParam`.

---

### 🟣 Nível 5: Disparando POST com Serialização Automática de JSON
**Cenário:** Você precisa enviar um payload JSON no corpo da requisição POST para cadastrar um novo usuário.
**Sua Tarefa:**
* Crie uma instância de `UserDetailsRequestModel details = new UserDetailsRequestModel("Lucas", "senha123");`.
* Monte a requisição informando o tipo de conteúdo e passando o objeto diretamente no método `.body()`:
  ```java
  given()
      .contentType(ContentType.JSON)
      .accept(ContentType.JSON)
      .body(details)
  .when()
      .post("/users")
  .then()
      .statusCode(200);
  ```
* Observe que o REST Assured serializa o objeto Java para JSON automaticamente sem necessidade de chamar o `ObjectMapper` manualmente.

---

### 🟤 Nível 6: Validando Propriedades do JSON com Hamcrest Matchers
**Cenário:** Além de checar o status HTTP, você precisa validar que o JSON retornado contém as propriedades e valores esperados.
**Sua Tarefa:**
* Importe os matchers estáticos de `org.hamcrest.Matchers.*` (como `equalTo`, `notNullValue`, `hasSize`).
* No bloco `.then()`, encadeie as asserções de corpo JsonPath:
  ```java
  .then()
      .statusCode(200)
      .body("firstName", equalTo("Lucas"))
      .body("userId", notNullValue());
  ```
* Tente alterar o valor do `equalTo` para ver a mensagem explicativa de incompatibilidade do Hamcrest.

---

### 🔵 Nível 7: Extraindo Dados da Resposta e Desserializando para Classes Java
**Cenário:** A validação é extensa ou você precisa salvar um token da resposta para ser utilizado em chamadas subsequentes.
**Sua Tarefa:**
* Interrompa a cadeia do REST Assured chamando `.extract().response()`:
  ```java
  Response response = given()
          .contentType(ContentType.JSON)
          .body(credenciais)
      .when()
          .post("/login")
      .then()
          .statusCode(200)
          .extract().response();
  ```
* Capture um cabeçalho: `String token = response.header("Authorization");`.
* Desserialize o JSON inteiro para uma classe Java: `UserRest user = response.as(UserRest.class);`.
* Valide com as asserções padrão do JUnit 5 (`assertEquals`, `assertNotNull`).

---

### 🟢 Nível 8: Depurando Transações com `.log().all()`
**Cenário:** Seu teste está falhando com status `415 Unsupported Media Type` ou `400 Bad Request` e você precisa inspecionar exatamente o que está trafegando na rede.
**Sua Tarefa:**
* Adicione `.log().all()` tanto no `given()` quanto no `then()`:
  ```java
  given()
      .log().all()
      ...
  .when()
      ...
  .then()
      .log().all()
      ...
  ```
* Execute o teste e observe no console a impressão completa dos headers enviados, payload formatado, status recebido e resposta JSON retornada pelo servidor.

---

### 🟡 Nível 9: Centralizando Configurações com `RequestSpecBuilder` e `ResponseSpecBuilder`
**Cenário:** Você tem 20 métodos de teste na mesma classe e está repetindo `.contentType(ContentType.JSON)` e filtros de log em todos eles.
**Sua Tarefa:**
* No método `@BeforeEach`, centralize os padrões utilizando especificações globais:
  ```java
  RestAssured.requestSpecification = new RequestSpecBuilder()
          .setContentType(ContentType.JSON)
          .addFilter(new RequestLoggingFilter())
          .addFilter(new ResponseLoggingFilter())
          .build();

  RestAssured.responseSpecification = new ResponseSpecBuilder()
          .expectResponseTime(lessThan(3000L))
          .build();
  ```
* Simplifique seus métodos `@Test` removendo configurações repetitivas e comprove que todos herdam o comportamento padrão.

---

### 🟠 Nível 10: Integração Final (Fluxo E2E de API com Autenticação e Rest Assured)
**Cenário:** Você precisa validar o ciclo de vida completo de uma API REST protegida por JWT utilizando puramente REST Assured.
**Sua Tarefa:**
* Configure a suíte com `@SpringBootTest(webEnvironment = RANDOM_PORT)`, `@TestInstance(PER_CLASS)` e `@TestMethodOrder(OrderAnnotation.class)`.
* Implemente:
  1. `@Test @Order(1)`: Dispara POST `/users` e valida a criação da conta.
  2. `@Test @Order(2)`: Dispara POST `/users/login`, extrai o token JWT do cabeçalho de resposta e armazena na variável da classe de teste.
  3. `@Test @Order(3)`: Dispara GET `/users/{id}` passando o token no header `Authorization: Bearer <token>` e valida se os dados retornam com sucesso (`200 OK`).
  4. `@Test @Order(4)`: Dispara DELETE `/users/{id}` com token e valida a remoção com status `204 No Content` ou `200 OK`.
