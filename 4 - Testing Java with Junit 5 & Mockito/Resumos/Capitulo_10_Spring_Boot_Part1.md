# Capítulo 10: Spring Boot (Parte 1) - Testando REST Controllers

Este capítulo introduz o mundo de testes em aplicações Spring Boot. Entenderemos a diferença entre rodar um teste de integração completo e testar fatias isoladas do sistema (neste caso, isolando apenas a camada de Rotas / Controllers).

## 1. Arquitetura de Camadas
Aplicações Spring Boot padrão seguem uma arquitetura em 3 camadas:
1.  **Web Layer:** Rest Controllers e mapeamento de rotas.
2.  **Service Layer:** Regras e lógicas de negócios complexas.
3.  **Data Layer:** Comunicação com Bancos de Dados (Repositories / DAOs).

Ao testar a Camada Web, queremos avaliar se as rotas (`@PostMapping`, `@GetMapping`) estão corretas, se recebemos parâmetros corretamente (JSON) e se a validação (Bean Validation) reage devolvendo o código HTTP certo. **Não** queremos misturar o banco de dados nesse momento.

## 2. Dependências
Ao criar um projeto no Spring Initializr, precisamos ter as seguintes bibliotecas:
*   **`spring-boot-starter-test`**: O pacote dourado. Já vem com o JUnit 5, o Mockito completo, o AssertJ e outras libs prontas.
*   **`spring-security-test`**: Necessário caso a aplicação utilize controle de acessos (Spring Security).
*   **`spring-boot-starter-validation`**: Necessário para habilitar as anotações (`@Valid`, `@Size`, `@Email`) nas nossas classes de DTO / Requests.

## 3. Teste Fatiado de Web (`@WebMvcTest`)
Para subir apenas o "mini servidor" focado apenas nos componentes da web (evitando carregar conexões de banco de dados e todos os outros serviços), utilizamos a anotação:

```java
@WebMvcTest(controllers = UsersController.class, excludeAutoConfiguration = {SecurityAutoConfiguration.class})
class UsersControllerTest { ... }
```
*(Aqui estamos isolando apenas o `UsersController` e desativando o filtro do Spring Security para focar puramente nas regras do endpoint).*

## 4. O Simulador de Requisições: `MockMvc`
Para invocar nossas rotas sem precisar de ferramentas como *Postman*, e sem subir de verdade um servidor Apache Tomcat, o Spring nos fornece um client poderoso chamado **`MockMvc`**.

Ele nos ajuda a "fingir" um envio HTTP. Para passar objetos JSON pra lá e pra cá, usamos o `ObjectMapper` da biblioteca Jackson.
```java
@Autowired
private MockMvc mockMvc; // Falso servidor web

@Test
void testaCricaoComMockMvc() throws Exception {
    // 1. Arrange: Montamos a classe Java de Request
    UserDetailsRequestModel details = new UserDetailsRequestModel("Lucas", "senha123");
    
    // 2. Transforma objeto Java em String JSON
    String jsonBody = new ObjectMapper().writeValueAsString(details);
    
    // 3. Monta uma simulação de requisição POST
    RequestBuilder builder = MockMvcRequestBuilders.post("/users")
            .contentType(MediaType.APPLICATION_JSON)
            .accept(MediaType.APPLICATION_JSON)
            .content(jsonBody);

    // 4. Act: Dispara o "send"
    MvcResult result = mockMvc.perform(builder).andReturn();
    
    // 5. Assert: Lemos o JSON que retornou da API e convertemos de volta pra Classe
    String jsonRetorno = result.getResponse().getContentAsString();
    UserRest objSalvo = new ObjectMapper().readValue(jsonRetorno, UserRest.class);
    
    assertEquals("Lucas", objSalvo.getFirstName());
}
```

## 5. Substituindo as Dependências do Controller com `@MockBean`
Como o `@WebMvcTest` não carrega as lógicas da Camada de Serviço, a Injeção de Dependências do Controller vai quebrar, porque ele espera receber um `UserService`.
A solução é usar **`@MockBean`** (em vez do `@Mock` tradicional do Mockito puro).

*   O `@MockBean` gera o dublê falso do Mockito **E TAMBÉM** adiciona esse falso objeto dentro do contexto oficial de injeção de dependência do Spring (`ApplicationContext`).
```java
@MockBean
UserService userService; // Agora o Controller conseguirá ser construído com esse Dublê

@Test
void exemplo() {
    when(userService.createUser(any())).thenReturn(meuDtoDeRetornoDesejado);
    // ... roda o mockMvc.perform(...)
}
```

## 6. Testando Cenários Negativos de Validação (`400 Bad Request`)
Se algum desenvolvedor apagar acidentalmente o `@Valid` do Controller, nossa API passará a aceitar e-mails em branco e usuários sem nome, quebrando o sistema.

A forma correta de testar essas proteções é enviar um JSON quebrando as regras e esperar um status de erro:
```java
@Test
void tentarSalvarComNomeVazioDeveRetornarBadRequest() throws Exception {
    // Simulamos um usuário inválido com nome vazio (quebrando o @Size(min=2))
    UserDetailsRequestModel details = new UserDetailsRequestModel("", "senha123");
    
    // ... Monta o builder com JSON e chama o mockMvc.perform() ...
    MvcResult result = mockMvc.perform(builder).andReturn();
    
    // Pega o código de status HTTP resultante
    int statusCode = result.getResponse().getStatus();
    
    // Compara se realmente o Spring barrou a requisição com código 400
    assertEquals(HttpStatus.BAD_REQUEST.value(), statusCode); 
}
```
