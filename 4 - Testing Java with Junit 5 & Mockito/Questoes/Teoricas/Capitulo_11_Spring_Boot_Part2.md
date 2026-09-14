# Questões Teóricas - Capítulo 11: Spring Boot (Parte 2) - Testes End-to-End (E2E)

Testes de fixação sobre testes de integração E2E com `@SpringBootTest`, portas dinâmicas, `TestRestTemplate`, autenticação com JWT e compartilhamento de estado com `PER_CLASS`.

---

### 1. Qual é a principal diferença conceitual entre os testes fatiados com `@WebMvcTest` e os testes de integração completos com `@SpringBootTest`?

> [!faq]- 👀 Ver Resposta
> Enquanto o `@WebMvcTest` isola exclusivamente os controllers da camada web utilizando componentes simulados em memória, o `@SpringBootTest` inicializa o ecossistema completo da aplicação (o `ApplicationContext` em sua totalidade). Ele carrega todos os beans de configuração, controllers, services, repositórios de dados reais ou em memória (como H2) e inicializa um servidor web embutido verdadeiro para testar o fluxo ponta a ponta da aplicação.

---

### 2. O que representam as configurações do parâmetro `webEnvironment` na anotação `@SpringBootTest` e qual a diferença entre `MOCK`, `DEFINED_PORT` e `RANDOM_PORT`?

> [!faq]- 👀 Ver Resposta
> O `webEnvironment` configura como o ambiente do servidor web será iniciado durante o teste:
> * **`WebEnvironment.MOCK` (Padrão):** Não sobe um servidor web HTTP real; utiliza um contêiner de servlet mockado (compatível com `MockMvc`).
> * **`WebEnvironment.DEFINED_PORT`:** Inicia um servidor web real (como Tomcat) na porta exata configurada no arquivo de propriedades (ex: porta 8080).
> * **`WebEnvironment.RANDOM_PORT`:** Inicia o servidor web real escutando em uma porta livre escolhida aleatoriamente pelo sistema operacional.

---

### 3. Por que a configuração `WebEnvironment.RANDOM_PORT` é amplamente recomendada para a realização de testes de integração?

> [!faq]- 👀 Ver Resposta
> Porque ela elimina o risco de erros de colisão de portas (*Address already in use / BindException*). Em ambientes de desenvolvimento compartilhado ou em servidores de Integração Contínua (CI/CD) onde múltiplos builds rodam simultaneamente na mesma máquina, fixar a porta 8080 causaria falhas de execução. A porta aleatória garante que cada processo de teste suba em uma porta exclusiva e permite paralelismo seguro.

---

### 4. Como o desenvolvedor pode descobrir dinamicamente qual porta TCP foi atribuída ao servidor na inicialização do `@SpringBootTest(webEnvironment = RANDOM_PORT)`?

> [!faq]- 👀 Ver Resposta
> Declarando um campo numérico na classe de teste anotado com a anotação **`@LocalServerPort`** (por exemplo, `@LocalServerPort private int port;`). O Spring Boot detecta a porta sorteada pelo container web em tempo de execução e a injeta diretamente na variável antes da execução dos testes.

---

### 5. Qual é o papel da anotação `@TestPropertySource` em classes de teste de integração do Spring Boot?

> [!faq]- 👀 Ver Resposta
> A anotação `@TestPropertySource(locations = "/application-test.properties")` permite carregar um arquivo de configurações de propriedades específico para o ambiente de testes. Isso viabiliza sobrescrever parâmetros de produção — como credenciais de conexão, URLs de banco de dados (apontando para H2 em vez do MySQL de produção), níveis de log ou portas — sem interferir no arquivo `application.properties` principal do sistema.

---

### 6. O que é o `TestRestTemplate` e em que ele se diferencia do `MockMvc`?

> [!faq]- 👀 Ver Resposta
> O `TestRestTemplate` é um cliente HTTP de alto nível fornecido pelo Spring Test para realizar requisições de rede verdadeiras contra um servidor HTTP real em execução (como no modo `RANDOM_PORT`). Diferente do `MockMvc`, que simula a requisição internamente no pipeline do servlet sem abrir conexões de rede, o `TestRestTemplate` envia pacotes TCP reais pela pilha de rede do sistema operacional, testando inclusive filtros de rede, certificados, serializadores e barreiras de segurança reais.

---

### 7. Como o método `exchange()` do `TestRestTemplate` auxilia no envio de requisições personalizadas com cabeçalhos HTTP?

> [!faq]- 👀 Ver Resposta
> Métodos simplificados como `getForObject()` ou `postForEntity()` não oferecem flexibilidade total para customizar todos os detalhes da transação. O método `exchange()` aceita a URI alvo, o verbo HTTP (`HttpMethod`), um objeto genérico `HttpEntity` (contendo tanto o corpo do payload quanto uma coleção customizada de `HttpHeaders`, como tokens Bearer de autenticação) e a classe de resposta (`Class<T>`), permitindo disparar qualquer combinação avançada de cabeçalhos e verbos.

---

### 8. Por que testes de integração que necessitam encadear operações (como Cadastro -> Login -> Acesso Protegido) falham ao armazenar o token em uma variável de classe se o ciclo de vida for o padrão do JUnit?

> [!faq]- 👀 Ver Resposta
> Porque o JUnit adota por padrão o ciclo de vida `TestInstance.Lifecycle.PER_METHOD`. A cada método `@Test` executado, uma nova instância da classe de teste é criada pelo framework e a instância anterior é descartada pelo Garbage Collector. Portanto, qualquer variável de instância (como um `String jwtToken`) populada pelo método de Login voltará a ser `null` no momento em que o método seguinte for executado em um novo objeto.

---

### 9. Como a anotação `@TestInstance(TestInstance.Lifecycle.PER_CLASS)` resolve o compartilhamento de tokens ou IDs entre etapas de um teste de integração?

> [!faq]- 👀 Ver Resposta
> Ao definir o ciclo de vida como `PER_CLASS`, o JUnit instancia a classe de teste **uma única vez** para todos os métodos de teste declarados. Dessa forma, atributos e variáveis de instância persistem na memória durante toda a suíte, permitindo que dados gerados em uma etapa anterior (como o token JWT obtido no login ou o ID do usuário cadastrado) fiquem disponíveis para os métodos subsequentes.

---

### 10. Por que o uso de `@TestInstance(Lifecycle.PER_CLASS)` torna indispensável o uso conjunto de `@TestMethodOrder(MethodOrderer.OrderAnnotation.class)` e `@Order(...)`?

> [!faq]- 👀 Ver Resposta
> Quando os testes dependem de estado compartilhado cumulativo (fluxos sequenciais dependentes onde o passo 3 precisa do resultado do passo 2), a ordem de execução se torna crítica. Como a ordem de execução padrão dos métodos no JUnit é indefinida e não determinística, sem o `@TestMethodOrder` o JUnit poderia tentar rodar o teste de "Acesso Protegido" antes do método de "Login", causando falhas aleatórias por falta do token que ainda não teria sido gerado.
