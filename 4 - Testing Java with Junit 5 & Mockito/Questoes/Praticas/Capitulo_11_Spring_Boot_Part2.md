# Questões Práticas - Capítulo 11 (Spring Boot Parte 2 - Testes End-to-End E2E)

---

### 🟢 Nível 1: Subindo o Servidor Real com `@SpringBootTest` e `RANDOM_PORT`
**Cenário:** Você precisa criar um teste de integração completo de ponta a ponta que inicialize o Tomcat embutido em uma porta dinâmica.
**Sua Tarefa:**
* Crie a classe de teste `UsuarioE2ETest`.
* Adicione no topo da classe a anotação:
  `@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)`.
* Declare uma variável inteira anotada com `@LocalServerPort private int port;`.
* Crie um `@Test` e imprima no console o valor de `this.port` para constatar que uma porta livre do sistema operacional foi alocada.

---

### 🟡 Nível 2: Sobrescrevendo Propriedades com `@TestPropertySource`
**Cenário:** Em produção, a aplicação conecta a um banco MySQL externo, mas na suíte E2E ela deve rodar com banco de testes e logs limpos.
**Sua Tarefa:**
* Crie o arquivo `src/test/resources/application-test.properties`.
* Adicione propriedades customizadas de teste (ex: `spring.jpa.show-sql=true` e `meu.parametro=ambiente_teste`).
* Na sua classe de teste, adicione `@TestPropertySource(locations = "/application-test.properties")`.
* Injete a propriedade com `@Value("${meu.parametro}") String param;` e assere que o valor carregado veio do arquivo de testes.

---

### 🟠 Nível 3: O Primeiro Disparo com `TestRestTemplate`
**Cenário:** Com o servidor rodando na porta dinâmica, você quer disparar uma requisição HTTP real contra o endpoint `/users`.
**Sua Tarefa:**
* Injete o cliente na classe: `@Autowired private TestRestTemplate restTemplate;`.
* Monte um objeto `UserDetailsRequestModel` com dados válidos.
* Dispare a requisição POST utilizando `postForEntity`:
  ```java
  ResponseEntity<UserRest> response = restTemplate.postForEntity(
          "/users", 
          meuRequestModel, 
          UserRest.class
  );
  ```
* Valide com `assertEquals(HttpStatus.OK, response.getStatusCode())` ou `HttpStatus.CREATED`.

---

### 🔴 Nível 4: Validando Bloqueio de Rota Protegida (Spring Security)
**Cenário:** A rota `/users` com verbo GET é restrita a usuários autenticados. Se uma chamada for feita sem cabeçalho de autenticação, o Spring Security deve barrá-la.
**Sua Tarefa:**
* Sem fornecer nenhum header de autorização, dispare `restTemplate.getForEntity("/users", String.class)`.
* Assere que a resposta recebida possui o status HTTP `HttpStatus.FORBIDDEN` (`403`) ou `UNAUTHORIZED` (`401`).
* Comprove que a segurança da aplicação está ativa nos testes de integração.

---

### 🟣 Nível 5: Realizando Login e Capturando o Token JWT
**Cenário:** O usuário foi cadastrado e agora precisa realizar login na rota `/users/login` enviando credenciais para receber o token de autenticação.
**Sua Tarefa:**
* Envie um POST para `/users/login` com o e-mail e senha cadastrados.
* Após o sucesso `200 OK`, acerte na resposta a leitura do cabeçalho `Authorization`:
  ```java
  String bearerToken = response.getHeaders().getValuesAsList("Authorization").get(0);
  ```
* Assere que o token não é nulo e começa com o prefixo `"Bearer "` ou com o padrão do seu sistema de tokens.

---

### 🟤 Nível 6: Diagnosticando a Perda de Estado no Ciclo de Vida Padrão
**Cenário:** Você declarou uma variável `private String token;` na classe de teste. No método `testLogin()` você salvou o token nela, mas no método seguinte `testAcessarRotaProtegida()`, a variável volta a ter o valor `null`.
**Sua Tarefa:**
* Explique por que o ciclo de vida padrão do JUnit (`PER_METHOD`) zera os atributos de instância entre um método de teste e outro.
* O que acontece com a execução do segundo teste se ele tentar usar a variável nula no cabeçalho HTTP?

---

### 🔵 Nível 7: Mantendo Estado Compartilhado com `@TestInstance(PER_CLASS)`
**Cenário:** Para resolver o problema do Nível 6 e viabilizar o compartilhamento do token JWT obtido no login, você precisa alterar o ciclo de vida da classe de testes.
**Sua Tarefa:**
* Adicione no topo da classe: `@TestInstance(TestInstance.Lifecycle.PER_CLASS)`.
* Declare o campo `private String authorizationToken;`.
* No método de login, salve o token: `this.authorizationToken = bearerToken;`.
* No método seguinte, verifique se `this.authorizationToken` continua guardando a string do token com sucesso.

---

### 🟢 Nível 8: Forçando a Sequência Determinística com `@Order`
**Cenário:** Com o estado compartilhado, se o método de consulta rodar antes do método de cadastro ou login, o teste quebrará por falta de dados no banco e ausência de token.
**Sua Tarefa:**
* Adicione no topo da classe: `@TestMethodOrder(MethodOrderer.OrderAnnotation.class)`.
* Atribua as ordens estritas aos métodos:
  * `@Test @Order(1) void testCadastrarUsuario()`
  * `@Test @Order(2) void testTentarAcessarSemToken_DeveRetornar403()`
  * `@Test @Order(3) void testLogin_DeveRetornarTokenJWT()`
  * `@Test @Order(4) void testAcessarComTokenValido_DeveRetornarLista()`
* Execute a classe inteira e certifique-se de que a sequência é executada rigorosamente do 1 ao 4.

---

### 🟡 Nível 9: Enviando Headers com `HttpHeaders` e `TestRestTemplate.exchange()`
**Cenário:** Com o token capturado no Passo 3, você precisa realizar uma requisição GET na rota protegida `/users` anexando o cabeçalho `Authorization: Bearer <token>`.
**Sua Tarefa:**
* Crie uma instância de `HttpHeaders`:
  ```java
  HttpHeaders headers = new HttpHeaders();
  headers.setAccept(Arrays.asList(MediaType.APPLICATION_JSON));
  headers.setBearerAuth(this.authorizationToken.replace("Bearer ", ""));
  ```
* Envolva os cabeçalhos em um `HttpEntity<Void> requestEntity = new HttpEntity<>(headers);`.
* Execute o disparo usando o método flexível `exchange()`:
  ```java
  ResponseEntity<UserRest[]> response = restTemplate.exchange(
          "/users", 
          HttpMethod.GET, 
          requestEntity, 
          UserRest[].class
  );
  ```
* Assere que o status retornado é `200 OK` e que a lista contém os dados do usuário.

---

### 🟠 Nível 10: Integração Final (Pipeline E2E Completo e Autenticado)
**Cenário:** Você precisa entregar uma suíte E2E automatizada cobrindo todo o ciclo de vida de uma conta de usuário em um ambiente de produção simulado.
**Sua Tarefa:**
* Construa a classe `FluxoCompletoUsuarioE2ETest` com `@SpringBootTest(webEnvironment = RANDOM_PORT)`, `@TestInstance(PER_CLASS)` e `@TestMethodOrder(OrderAnnotation.class)`.
* Conecte e encadeie os 5 passos ordenados:
  1. `@Order(1)`: Cria o usuário via POST e valida se o banco persistiu o registro.
  2. `@Order(2)`: Tenta acessar `/users/me` sem token e valida a rejeição com `403 Forbidden`.
  3. `@Order(3)`: Executa o POST `/login`, extrai o JWT e armazena no atributo da classe.
  4. `@Order(4)`: Realiza GET `/users/me` utilizando o token Bearer e valida as informações do perfil retornado.
  5. `@Order(5)`: Atualiza o nome do usuário via PUT autenticado e confirma se uma nova busca reflete o novo nome.
