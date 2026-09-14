# Questões Práticas - Capítulo 10 (Spring Boot Parte 1 - Testando REST Controllers)

---

### 🟢 Nível 1: Configurando o Teste Fatiado de Web com `@WebMvcTest`
**Cenário:** Você precisa testar o endpoint `UsersController` sem carregar o banco de dados nem os serviços pesados da aplicação Spring Boot.
**Sua Tarefa:**
* Crie a classe de teste `UsersControllerTest`.
* Anote a classe com `@WebMvcTest(controllers = UsersController.class)`.
* Adicione a exclusão da segurança para focar apenas nas regras web:
  `excludeAutoConfiguration = {SecurityAutoConfiguration.class}`.
* Injete a ferramenta `MockMvc` utilizando a anotação `@Autowired private MockMvc mockMvc;`.

---

### 🟡 Nível 2: Injetando Dublês no Contexto Spring com `@MockBean`
**Cenário:** O seu `UsersController` recebe a dependência `UserService` em seu construtor. Como o `@WebMvcTest` não carrega classes `@Service`, o contexto falha ao subir.
**Sua Tarefa:**
* Declare um campo na classe de teste anotado com `@MockBean private UserService userService;`.
* Explique por que a anotação `@MockBean` do Spring Boot foi necessária neste cenário em vez do `@Mock` tradicional do Mockito.
* Execute o teste e confirme que o `ApplicationContext` inicializa com sucesso.

---

### 🟠 Nível 3: Serializando Objetos em JSON com `ObjectMapper`
**Cenário:** Para disparar uma requisição POST simulada com corpo JSON, você precisa serializar uma instância de DTO em texto puro.
**Sua Tarefa:**
* Crie uma instância de `UserDetailsRequestModel details = new UserDetailsRequestModel("Lucas", "senha123");`.
* Instancie ou injete o `ObjectMapper` da biblioteca Jackson.
* Converta o objeto para JSON utilizando `String jsonBody = new ObjectMapper().writeValueAsString(details);`.
* Imprima a string e verifique a conformidade da estrutura JSON gerada.

---

### 🔴 Nível 4: Disparando a Requisição POST com `MockMvcRequestBuilders`
**Cenário:** Você quer simular o envio de uma requisição HTTP POST para a rota `/users` enviando headers e payload JSON.
**Sua Tarefa:**
* Monte a requisição usando a API fluente do `MockMvcRequestBuilders`:
  ```java
  RequestBuilder requestBuilder = MockMvcRequestBuilders.post("/users")
          .contentType(MediaType.APPLICATION_JSON)
          .accept(MediaType.APPLICATION_JSON)
          .content(jsonBody);
  ```
* Dispare a chamada com `MvcResult result = mockMvc.perform(requestBuilder).andReturn();`.
* Imprima o status HTTP da resposta através de `result.getResponse().getStatus()`.

---

### 🟣 Nível 5: Condicionando o Retorno do `@MockBean` (Stubbing)
**Cenário:** Ao disparar o POST, o controller invoca `userService.createUser(...)`. Se o mock retornar `null`, o controller responderá com erro ou corpo vazio.
**Sua Tarefa:**
* Antes de disparar a requisição com `mockMvc`, configure o comportamento do mock na etapa Arrange:
  ```java
  UserRest userRetornado = new UserRest("USER-123", "Lucas", "lucas@email.com");
  when(userService.createUser(any(UserDto.class))).thenReturn(userRetornado);
  ```
* Execute a chamada via `mockMvc.perform(...)` e certifique-se de que o controller consegue processar o retorno do serviço dublado.

---

### 🟤 Nível 6: Desserializando a Resposta e Asserindo o DTO Retornado
**Cenário:** Você precisa inspecionar os dados textuais JSON retornados na resposta HTTP e convertê-los de volta para um objeto Java para fazer asserções tipadas.
**Sua Tarefa:**
* Extraia a String de resposta através de `String jsonResponse = result.getResponse().getContentAsString();`.
* Desserialize o JSON utilizando o Jackson:
  ```java
  UserRest usuarioCriado = new ObjectMapper().readValue(jsonResponse, UserRest.class);
  ```
* Assere com JUnit 5 que `usuarioCriado.getUserId()` é igual a `"USER-123"` e que o nome é `"Lucas"`.

---

### 🔵 Nível 7: Testando Requisições GET com Parâmetros de Rota
**Cenário:** Você precisa testar o endpoint `GET /users/{userId}` que busca os detalhes de um usuário cadastrado.
**Sua Tarefa:**
* Configure o `when(userService.getUserByUserId("USER-999"))` para retornar um `UserRest`.
* Dispare a requisição GET:
  ```java
  MvcResult result = mockMvc.perform(MockMvcRequestBuilders.get("/users/{userId}", "USER-999")
          .accept(MediaType.APPLICATION_JSON))
          .andReturn();
  ```
* Valide se o status de retorno é `200 OK` e se o corpo contém as propriedades corretas do usuário.

---

### 🟢 Nível 8: Testando Validações Negativas (Bean Validation)
**Cenário:** O DTO `UserDetailsRequestModel` possui a restrição de validação `@NotBlank(message = "O primeiro nome é obrigatório")` no atributo `firstName`.
**Sua Tarefa:**
* Crie uma instância do DTO com `firstName` em branco ou nulo: `new UserDetailsRequestModel("", "senha123")`.
* Converta o objeto para JSON e envie a requisição via `MockMvcRequestBuilders.post("/users")`.
* Capture o status HTTP resultante com `result.getResponse().getStatus()`.
* Assere que o código devolvido pelo Spring é rigorosamente `HttpStatus.BAD_REQUEST.value()` (`400`).

---

### 🟡 Nível 9: Comprovando a Proteção contra Remoção de `@Valid`
**Cenário:** Para entender a importância do teste criado no Nível 8, você quer simular o erro de um desenvolvedor desatento.
**Sua Tarefa:**
* No código de produção do `UsersController`, remova temporariamente a anotação `@Valid` do argumento `@RequestBody UserDetailsRequestModel details`.
* Execute o teste unitário negativo construído no Nível 8.
* O teste falha acusando que recebeu `200 OK` em vez de `400 Bad Request`?
* Restaure a anotação `@Valid` no controller e observe o teste voltar a ficar verde.

---

### 🟠 Nível 10: Integração Final (Suíte Completa de Testes de Controller)
**Cenário:** Você foi encarregado de entregar a suíte de testes oficial do `ProdutosController`, validando múltiplos métodos e regras de integridade.
**Sua Tarefa:**
* Na classe `ProdutosControllerTest`, utilize `@WebMvcTest(controllers = ProdutosController.class)` com `@MockBean private ProdutoService produtoService`.
* Implemente três cenários exaustivos:
  1. **POST /produtos (Sucesso):** Envia payload válido, mock responde com produto cadastrado com ID `10`, valida status `201 Created` ou `200 OK` e dados desserializados.
  2. **POST /produtos (Preço Negativo):** Envia payload com preço `-5.0` (violando `@Positive`), valida status `400 Bad Request` e com `verify(produtoService, never()).criarProduto(any())` comprova que o serviço nem chegou a ser acionado.
  3. **GET /produtos/999 (Não Encontrado):** Mock lança ou retorna vazio, controller responde `404 Not Found`.
