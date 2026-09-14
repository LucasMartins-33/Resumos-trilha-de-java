# Questões Teóricas - Capítulo 10: Spring Boot (Parte 1) - Testando REST Controllers

Testes de fixação sobre arquitetura em camadas, testes fatiados com `@WebMvcTest`, simulação HTTP com `MockMvc`, injeção de dublês com `@MockBean` e Bean Validation.

---

### 1. No modelo arquitetural em 3 camadas do Spring Boot (Web, Service e Data), qual é o escopo e o propósito principal de testar a Camada Web (Controllers)?

<details>
<summary>👀 Ver Resposta</summary>

O propósito de testar a Camada Web é validar exclusivamente a "porta de entrada" da aplicação: garantir que as rotas e verbos HTTP (`@GetMapping`, `@PostMapping`, etc.) estão mapeados corretamente, validar a serialização e desserialização de payloads JSON, verificar o comportamento de anotações de validação (`@Valid`) e checar se os cabeçalhos e códigos de status HTTP corretos (ex: `200`, `201`, `400`, `404`) são devolvidos, mantendo as regras de negócio e o banco de dados fora do escopo do teste.
</details>

---

### 2. Quais bibliotecas essenciais para a automação de testes estão incluídas automaticamente pelo starter `spring-boot-starter-test`?

<details>
<summary>👀 Ver Resposta</summary>

O `spring-boot-starter-test` inclui:
* **JUnit Jupiter (JUnit 5):** O framework e engine de execução de testes.
* **Spring Test & Spring Boot Test:** Utilitários para inicialização e suporte ao contexto do Spring.
* **Mockito:** Para criação e gestão de dublês de teste (mocks e spies).
* **AssertJ:** Biblioteca para escrita de asserções fluentes.
* **JSONassert & JsonPath:** Ferramentas para inspecionar e validar documentos JSON.
</details>

---

### 3. O que é um "Teste Fatiado" (Slice Test) no Spring Boot e qual o papel da anotação `@WebMvcTest`?

<details>
<summary>👀 Ver Resposta</summary>

Um teste fatiado carrega apenas um subconjunto restrito do `ApplicationContext` do Spring, específico para a camada sob verificação, em vez de subir todo o ecossistema da aplicação. A anotação `@WebMvcTest` inicializa apenas componentes relacionados ao Spring MVC (como `@Controller`, `@ControllerAdvice`, filtros e conversores JSON), ignorando completamente componentes de serviço (`@Service`), componentes de persistência (`@Repository`) e conexões de banco de dados, tornando os testes muito mais rápidos.
</details>

---

### 4. O que é o utilitário `MockMvc` e qual a sua principal vantagem na execução de testes da camada web?

<details>
<summary>👀 Ver Resposta</summary>

O `MockMvc` é uma classe do Spring Test que provê suporte para testar controllers simulando todo o processamento de requisições e respostas HTTP dentro de um contêiner web falso em memória (mock servlet environment). Sua principal vantagem é permitir testar rotas, filtros, headers e validações sem o custo de inicializar um servidor web embutido real (como o Tomcat ou Jetty) e sem abrir sockets de rede verdadeiros, proporcionando altíssima velocidade de execução.
</details>

---

### 5. Qual é o papel da classe `ObjectMapper` (Jackson) nos testes que interagem com o `MockMvc`?

<details>
<summary>👀 Ver Resposta</summary>

O `ObjectMapper` realiza a conversão bidirecional entre objetos Java e documentos JSON:
* Na etapa Arrange/Act, ele serializa objetos DTO de requisição em formato de texto JSON (`writeValueAsString`) para serem enviados no corpo da requisição simulada via `MockMvcRequestBuilders`.
* Na etapa Assert, ele desserializa o corpo de resposta recebido (`getContentAsString`) de volta para instâncias de classes Java de resposta (`readValue`), facilitando a validação de atributos com asserções tipadas.
</details>

---

### 6. Qual a diferença crucial entre a anotação `@MockBean` do Spring Boot e a anotação `@Mock` do Mockito tradicional?

<details>
<summary>👀 Ver Resposta</summary>

* **`@Mock` (Mockito puro):** Cria um dublê de teste isolado na memória, sem qualquer ciência sobre o ecossistema do Spring.
* **`@MockBean` (Spring Boot Test):** Cria um dublê do Mockito e **adiciona ou substitui explicitamente esse bean dentro do `ApplicationContext` do Spring**. Assim, quando outros beans gerenciados pelo Spring (como um `@RestController`) exigirem a injeção daquela dependência, o Spring injetará o dublê configurado no contexto, viabilizando testes integrados ao container de injeção de dependências.
</details>

---

### 7. Por que a anotação `@MockBean` é frequentemente necessária em testes anotados com `@WebMvcTest`?

<details>
<summary>👀 Ver Resposta</summary>

Como o `@WebMvcTest` carrega apenas a camada web e ignora as classes anotadas com `@Service` e `@Repository`, qualquer controller que dependa de um serviço via injeção de dependência (`@Autowired` ou via construtor) falhará na inicialização do contexto com erro de "NoSuchBeanDefinitionException". A inclusão de `@MockBean MeuService meuService;` resolve essa dependência fornecendo um dublê simulado para o Spring satisfazer a injeção do controller.
</details>

---

### 8. Por que é fundamental escrever testes que forcem respostas `400 Bad Request` na camada de Controllers?

<details>
<summary>👀 Ver Resposta</summary>

Para certificar que os validadores automáticos do Spring (Bean Validation) e anotações como `@Valid`, `@NotBlank`, `@Size` ou `@Email` nos DTOs de entrada continuam ativos e configurados. Se um desenvolvedor remover acidentalmente a anotação `@Valid` dos parâmetros do método no controller, a API passará a aceitar dados inválidos e corrompidos sem rejeitá-los. Testes negativos garantem a preservação dessa barreira de integridade.
</details>

---

### 9. Qual é o propósito do parâmetro `excludeAutoConfiguration = {SecurityAutoConfiguration.class}` na anotação `@WebMvcTest`?

<details>
<summary>👀 Ver Resposta</summary>

Por padrão, se a dependência do Spring Security estiver presente no projeto, ela aplicará filtros obrigatórios de autenticação e proteção CSRF em todas as rotas web, fazendo com que requisições de teste simples recebam erros `401 Unauthorized` ou `403 Forbidden`. Desabilitar a autoconfiguração de segurança via `excludeAutoConfiguration` remove essa camada protetora temporariamente durante o teste unitário fatiado, permitindo focar os testes puramente nas regras funcionais do controller.
</details>

---

### 10. Como o método `andReturn()` do `MockMvc` difere de asserções encadeadas com `andExpect()`?

<details>
<summary>👀 Ver Resposta</summary>

O `andExpect(MockMvcResultMatchers...)` realiza asserções declarativas diretamente na cadeia de execução do MockMvc (por exemplo, `andExpect(status().isOk())`). Já o método `andReturn()` encerra a cadeia e devolve o objeto concreto `MvcResult`, contendo a requisição simulada e a resposta completa (`getResponse()`). O `andReturn()` é utilizado quando o desenvolvedor prefere extrair manualmente dados da resposta (como headers, corpo em texto ou status) para realizar asserções com métodos tradicionais do JUnit 5 (`assertEquals`) ou inspecionar objetos desserializados.
</details>
