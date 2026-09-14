# Questões Teóricas - Capítulo 15: Testando APIs RESTful com REST Assured

Testes de fixação sobre REST Assured, testes de API de caixa preta, sintaxe BDD Given-When-Then, Hamcrest matchers, extração de objetos e especificações globais.

---

### 1. O que é o REST Assured e qual o seu principal objetivo na automação de testes de aplicações corporativas?

> [!faq]- 👀 Ver Resposta
> O REST Assured é uma biblioteca Java desenvolvida para simplificar a automação de testes e validação de serviços web RESTful baseados em HTTP e JSON/XML. Seu objetivo primordial é fornecer uma DSL (Domain-Specific Language) elegante e legível para enviar requisições de rede verdadeiras e asserir de forma expressiva o status, cabeçalhos e conteúdos de resposta das APIs.

---

### 2. Qual é a diferença fundamental entre testar endpoints com `MockMvc` e testar endpoints utilizando o REST Assured?

> [!faq]- 👀 Ver Resposta
> O `MockMvc` atua dentro de um ambiente simulado de Servlet em memória, sem trafegar dados pela rede nem subir um servidor HTTP real, o que pode mascarar erros causados por firewalls, proxies, regras de CORS, timeouts de socket e filtros de servidores web reais. O REST Assured, por outro lado, opera como um cliente HTTP externo completo que trafega requisições pela camada de transporte de rede (TCP/IP) contra uma aplicação rodando de verdade, validando a pilha de ponta a ponta.

---

### 3. Por que se diz que o REST Assured permite realizar testes de "Caixa Preta" (Black-box testing)?

> [!faq]- 👀 Ver Resposta
> Porque o REST Assured não precisa de nenhum acesso ao código-fonte interno, classes Java ou contexto de injeção de dependências do servidor. Ele enxerga a aplicação exclusivamente através de seus contratos públicos de interface HTTP (rotas, verbos, headers e payloads JSON). Devido a esse desacoplamento total, o REST Assured escrito em Java pode ser utilizado para testar sistemas desenvolvidos em qualquer outra linguagem ou tecnologia (como Node.js, Python, Go ou .NET).

---

### 4. Explique a sintaxe fluente baseada em BDD utilizada pelo REST Assured: `given()`, `when()` e `then()`.

> [!faq]- 👀 Ver Resposta
> * **`given()` (Dado que... / Arrange):** Configura todos os pré-requisitos da requisição, como autenticação (tokens Bearer ou Basic Auth), cabeçalhos HTTP, parâmetros de rota, query parameters e o corpo (payload) enviado.
> * **`when()` (Quando... / Act):** Define a ação de disparo do verbo HTTP correspondente (`get()`, `post()`, `put()`, `delete()`) apontando para a URI do recurso.
> * **`then()` (Então... / Assert):** Encadeia as regras de validação da resposta, auditando códigos de status HTTP, headers recebidos, tempo de resposta e valores internos do JSON.

---

### 5. Como o REST Assured é configurado para apontar para a porta aleatória (`RANDOM_PORT`) de uma aplicação Spring Boot?

> [!faq]- 👀 Ver Resposta
> Em um método anotado com `@BeforeEach`, captura-se o número da porta TCP sorteada pelo Spring através da anotação `@LocalServerPort private int port;` e atribui-se esses dados às variáveis de configuração estáticas globais do framework:
> ```java
> RestAssured.baseURI = "http://localhost";
> RestAssured.port = port;
> ```
> A partir dessa configuração, todas as chamadas `when().get("/users")` direcionarão suas requisições automaticamente para o endereço e porta corretos.

---

### 6. Qual a diferença conceitual entre `pathParam()` e `queryParam()` no REST Assured?

> [!faq]- 👀 Ver Resposta
> * **`pathParam("id", 123)`:** Substitui marcadores dinâmicos contidos no caminho da própria rota da URL, como transformar `/users/{id}` em `/users/123`. É utilizado para identificar univocamente um recurso específico.
> * **`queryParam("page", 1)`:** Concatena parâmetros de consulta ao final da URL através da sintaxe de interrogação e chave-valor (ex: `/users?page=1&size=10`). É utilizado para filtragem, paginação, busca e ordenação de recursos.

---

### 7. Como o REST Assured integra-se com a biblioteca Hamcrest para validar propriedades internas de uma resposta JSON?

> [!faq]- 👀 Ver Resposta
> O método `.body("caminho.propriedade", Matcher)` utiliza o mecanismo de navegação JsonPath combinado com asserções declarativas do Hamcrest (como `equalTo()`, `hasSize()`, `containsString()`, `notNullValue()`). Por exemplo, a instrução `.body("firstName", equalTo("Lucas"))` navega até o nó `firstName` do documento JSON retornado e assere que o seu valor textual é estritamente igual a "Lucas".

---

### 8. Em quais situações é vantajoso utilizar o método `.extract().response()` em vez de realizar todas as validações dentro do bloco `then()`?

> [!faq]- 👀 Ver Resposta
> Quando a lógica de validação é muito complexa, envolve cálculos matemáticos avançados, comparações entre múltiplos nós do JSON, ou quando é necessário reaproveitar dados recebidos (como salvar um token JWT do cabeçalho de resposta em uma variável para o próximo teste). A extração converte a resposta em um objeto `Response` ou a desserializa para uma classe Java (`response.as(UserRest.class)`), permitindo inspecionar livremente seus atributos com asserções clássicas do JUnit 5.

---

### 9. Qual é a finalidade dos comandos de depuração `.log().all()` no REST Assured?

> [!faq]- 👀 Ver Resposta
> Os comandos `.log().all()` instruem o REST Assured a imprimir no console do terminal todos os detalhes técnicos da transação HTTP. Se invocado dentro de `given()`, ele exibe os cabeçalhos, parâmetros e payload que estão sendo enviados para o servidor. Se invocado dentro de `then()`, ele imprime os cabeçalhos de retorno, o status HTTP e o corpo completo retornado pelo servidor, facilitando a identificação de erros de formatação JSON e respostas de erro 4xx/5xx.

---

### 10. Para que servem as classes `RequestSpecification` e `ResponseSpecification` (`RequestSpecBuilder` / `ResponseSpecBuilder`) no REST Assured?

> [!faq]- 👀 Ver Resposta
> Servem para evitar a duplicação excessiva de código de configuração (DRY - *Don't Repeat Yourself*). Com o `RequestSpecBuilder`, o desenvolvedor define em um método centralizado `@BeforeEach` especificações compartilhadas por todos os testes (como cabeçalhos `Content-Type: application/json`, autenticações padrão e filtros de log). Da mesma forma, o `ResponseSpecBuilder` centraliza regras comuns de validação para todas as respostas, como exigir que nenhuma requisição ultrapasse um tempo limite máximo de resposta (*expectResponseTime*).
