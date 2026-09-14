# Capítulo 15: Testando APIs RESTful com REST Assured

Neste capítulo fomos apresentados a uma das ferramentas mais importantes do mercado para testes de integração de ponta-a-ponta (E2E): o **REST Assured**.

## 1. O que é e Por Que usar?
Diferente do `MockMvc` do Spring, que funciona através de simulações virtuais em um ambiente Web Falso, o REST Assured envia requisições HTTP reais de verdade pela rede. 
*   **Ponto Forte:** Se a sua API tem problemas reais de Cors, filtros de Segurança ou Firewalls, o REST Assured vai bater contra essas barreiras e falhar os testes com exatidão. O `MockMvc` mascararia isso.
*   **Caixa Preta:** Como ele realiza chamadas HTTP externas, você poderia usá-lo para testar APIs feitas em Node.js ou C#, sem precisar estar amarrado ao contexto do Spring.

## 2. A Sintaxe Fluente (BDD): Given, When, Then
O REST Assured utiliza uma sintaxe declarativa onde os métodos são encadeados (Fluent API), tornando a leitura semelhante à língua inglesa natural. Ele reflete com perfeição o padrão *Arrange-Act-Assert*:

*   **`given()` (Arrange):** Preparação da Requisição (Cabeçalhos, Autenticação, Parâmetros e Corpo).
*   **`when()` (Act):** Disparo da Requisição HTTP (Definição de Método GET/POST/PUT/DELETE e a URL).
*   **`then()` (Assert):** Validações da Resposta (Códigos de Status, Conteúdo do JSON, Headers).

## 3. Configuração Inicial e Amarração com o Spring Boot
Para que o REST Assured saiba onde a sua aplicação subiu (visto que usamos `RANDOM_PORT`), precisamos injetar e apontar a porta nas configurações globais estáticas do Rest Assured no `@BeforeEach`:

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class IntegrationTest {

    @LocalServerPort
    private int port;

    @BeforeEach
    void setupGlobal() {
        RestAssured.baseURI = "http://localhost";
        RestAssured.port = port;
    }
}
```

## 4. O Fluxo de Trabalho (Exemplo Prático Completo)

No exemplo abaixo construímos uma requisição avançada que insere QueryParams (`?page=1`), um PathParam (`/users/123`), passa o Token Bearer JWT e envia um JSON.

```java
@Test
void exemploCompletoDeRequisicaoEValidacao() {
    // 1. PREPARAÇÃO (GIVEN)
    given()
        .contentType(ContentType.JSON)                // Configura Headers Padrões
        .accept(ContentType.JSON)
        .header("Authorization", "Bearer " + token)   // Header de Autenticação
        .pathParam("id", "CODIGO-123")                // Define as variáveis de rota
        .queryParam("page", 1)                        // Cria ?page=1
        .body(meuObjetoRequest)                       // Rest Assured converte p/ JSON auto!

    // 2. DISPARO (WHEN)
    .when()
        .get("/users/{id}")                           // Usa o PathParam "id" na rota

    // 3. VALIDAÇÃO (THEN)
    .then()
        .statusCode(200)                              // Exige que responda 200 OK
        .body("firstName", equalTo("Lucas"))          // Valida a chave 'firstName' dentro do JSON de Resposta usando biblioteca Hamcrest!
        .body("size()", equalTo(10));                 // Checa se é um Array JSON e tem tamanho 10
}
```

## 5. Extraindo Valores Livres da Resposta
Muitas vezes, fazer as validações encadeadas com o `.body("chave", equalTo("valor"))` pode ser difícil de ler se o JSON for complexo. Podemos então parar a corrente do Rest Assured no `then()`, extrair o objeto e voltar a usar as Asserções Clássicas do JUnit:

```java
@Test
void extraindoEValidandoUsandoJUnitClasico() {
    // Retorna a resposta ao invés de prosseguir na corrente
    Response response = given()
            .body(credenciais)
        .when()
            .post("/login")
        .then()
            .statusCode(200)
            .extract().response(); // <--- Extração

    // O objeto "response" tem métodos pra ler e testar qualquer coisa!
    String token = response.header("Authorization");
    assertNotNull(token);

    // Converte o JSON inteiro de volta para a sua classe Java 
    UserRest user = response.as(UserRest.class);
    assertEquals("Lucas", user.getFirstName());
}
```

## 6. Solução de Problemas: Mostrando os Logs!
Testes E2E falham frequentemente por Headers errados ou JSONs mal formatados. O REST Assured possui os injetores de log para te ajudar a debugar a transação HTTP. Basta inserir o **`.log().all()`**:

```java
    given()
        .log().all() // Pinta no console todos os Headers e o Corpo que você ESTÁ ENVIANDO!
    .when()
        ...
    .then()
        .log().all() // Pinta no console tudo O QUE VOLTOU do Servidor (HTML, Erros 500, Headers...)
```

## 7. Eliminando Código Repetido com Specifications
Para não ter que repetir os Headers `ContentType.JSON` e o `.log().all()` em todos os 50 testes da sua classe, o framework nos permite centralizar essas obrigações em Especificações Globais:

```java
@BeforeEach
void configureSpecifications() {
    // Toda requisição criada por essa classe (GIVEN) já virá com ContentType JSON e Logs
    RestAssured.requestSpecification = new RequestSpecBuilder()
            .setContentType(ContentType.JSON)
            .addFilter(new RequestLoggingFilter())
            .addFilter(new ResponseLoggingFilter())
            .build();

    // Toda resposta processada (THEN) verificará se a API não demorou mais que 2 segundos
    RestAssured.responseSpecification = new ResponseSpecBuilder()
            .expectResponseTime(lessThan(2000L))
            .build();
}
```
