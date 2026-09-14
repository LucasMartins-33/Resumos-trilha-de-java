# Questões Teóricas - Capítulo 01: Introduction

Testes de fixação sobre conceitos fundamentais de testes unitários, boas práticas e arquitetura do JUnit 5.

---

### 1. O que caracteriza conceitualmente um teste unitário e qual é o seu principal objetivo no desenvolvimento de software?

> [!faq]- 👀 Ver Resposta
> Um teste unitário é um método automatizado focado em verificar o comportamento da menor unidade testável de uma aplicação (geralmente um método individual de uma classe) de forma totalmente isolada. Seu objetivo principal é garantir que a lógica de negócio execute exatamente como esperado sob condições específicas, provendo feedback imediato sobre defeitos antes de integrar com outros componentes ou serviços externos.

---

### 2. Explique a estrutura AAA (Arrange, Act, Assert) utilizada na escrita de testes unitários e o propósito de cada etapa.

> [!faq]- 👀 Ver Resposta
> A estrutura AAA padroniza a organização lógica do teste em três etapas consecutivas:
> * **Arrange (Preparar):** Configura o cenário de teste, instanciando os objetos necessários, preparando os dados de entrada e definindo o comportamento de dependências simuladas (mocks).
> * **Act (Agir):** Executa o método específico que está sob teste, geralmente capturando o valor retornado ou acionando a transição de estado.
> * **Assert (Afirmar/Validar):** Compara o resultado real obtido com o resultado esperado, garantindo que as regras de negócio foram satisfeitas.

---

### 3. O que é o acrônimo F.I.R.S.T. no contexto de testes automatizados e o que cada letra representa?

> [!faq]- 👀 Ver Resposta
> F.I.R.S.T. é um conjunto de boas práticas essenciais para testes unitários eficazes:
> * **Fast (Rápido):** Devem rodar em milissegundos para serem executados frequentemente sem atrapalhar o fluxo de trabalho.
> * **Independent (Independente):** Não devem compartilhar estado mutável nem depender da ordem de execução de outros testes.
> * **Repeatable (Repetível):** Devem produzir o mesmo resultado (passar ou falhar) em qualquer ambiente e a qualquer momento, sem depender de recursos voláteis.
> * **Self-validating (Autovalidável):** O teste deve determinar automaticamente seu sucesso ou falha (booleano), sem necessidade de inspeção manual de logs ou saídas de console.
> * **Thorough / Timely (Abrangente / Oportuno):** Devem cobrir casos felizes, exceções e valores limite, preferencialmente sendo escritos no momento ou antes do código de produção (como no TDD).

---

### 4. Por que testes unitários não devem realizar chamadas de rede ou acessar bancos de dados reais?

> [!faq]- 👀 Ver Resposta
> Porque chamadas de rede e acessos a banco de dados violam os princípios fundamentais de **Fast** e **Repeatable**. Operações de E/S (disco e rede) são ordens de magnitude mais lentas que operações em memória e introduzem dependências de infraestrutura externa. Se a rede oscilar ou o banco de dados cair, o teste falhará devido a fatores de infraestrutura e não por um defeito real no código de negócio da unidade testada.

---

### 5. O que é regressão no ciclo de vida de um software e como os testes unitários auxiliam a preveni-la?

> [!faq]- 👀 Ver Resposta
> Regressão é o reaparecimento de bugs ou a quebra acidental de funcionalidades existentes provocada por novas alterações, correções ou refatorações de código. Uma suíte de testes unitários automatizada funciona como uma rede de segurança: sempre que o desenvolvedor altera uma parte do código, a execução dos testes garante imediatamente que os comportamentos previamente consolidados continuam funcionando.

---

### 6. Como a Injeção de Dependências (Dependency Injection) facilita a testabilidade de uma classe?

> [!faq]- 👀 Ver Resposta
> Ao aplicar Injeção de Dependências, a classe sob teste recebe suas dependências externamente (geralmente via construtor), em vez de instanciá-las rigidamente com o operador `new` dentro de si. Isso desacopla as classes e permite que, no ambiente de teste unitário, as dependências reais (como repositórios de banco ou serviços de envio de e-mail) sejam facilmente substituídas por dublês de teste (mocks ou stubs), isolando a unidade sob verificação.

---

### 7. Explique a Pirâmide de Testes (Testing Pyramid) e a proporção recomendada entre seus níveis (Unitários, Integração e End-to-End).

> [!faq]- 👀 Ver Resposta
> A Pirâmide de Testes é um modelo conceitual que orienta a distribuição da cobertura de testes em um sistema:
> * **Base (Testes Unitários):** Maior quantidade. São rápidos, baratos de escrever e manter, e altamente focados em isolamento.
> * **Meio (Testes de Integração):** Quantidade intermediária. Validam a interação e a comunicação correta entre módulos da aplicação e serviços externos (banco, mensageria, etc.). São mais lentos e custosos que os unitários.
> * **Topo (Testes E2E / Ponta a Ponta):** Menor quantidade. Testam o sistema completo simulando o fluxo real do usuário. São os mais lentos, frágeis e caros de executar e manter.

---

### 8. Em relação à arquitetura modular do JUnit 5, quais são os seus três componentes fundamentais e qual é o papel do JUnit Platform?

> [!faq]- 👀 Ver Resposta
> O JUnit 5 é dividido em:
> 1. **JUnit Platform:** A base da arquitetura. Fornece a infraestrutura para inicializar e executar frameworks de teste na JVM (Java Virtual Machine), definindo a interface `TestEngine` que é consumida por IDEs (IntelliJ, Eclipse) e ferramentas de build (Maven, Gradle).
> 2. **JUnit Jupiter:** O modelo de programação e extensão moderno para escrever testes no JUnit 5.
> 3. **JUnit Vintage:** Módulo de suporte e retrocompatibilidade para executar testes legados escritos em JUnit 3 e JUnit 4.

---

### 9. Qual é o papel do JUnit Jupiter no ecossistema do JUnit 5 e como ele se diferencia do JUnit Vintage?

> [!faq]- 👀 Ver Resposta
> O **JUnit Jupiter** é a API moderna que provê as novas anotações, asserções e o modelo de extensão do JUnit 5 (como as anotações do pacote `org.junit.jupiter.api.*`, por exemplo `@Test`, `@BeforeEach`, `@DisplayName`). Já o **JUnit Vintage** é um motor legado (`VintageTestEngine`) cujo único propósito é garantir que projetos antigos com testes baseados em JUnit 3 ou JUnit 4 continuem sendo executados normalmente sobre a nova JUnit Platform sem necessidade de reescrita imediata.

---

### 10. Por que os testes unitários são considerados uma forma de "documentação viva" de um sistema?

> [!faq]- 👀 Ver Resposta
> Ao contrário de documentações em texto estático que rapidamente ficam desatualizadas em relação à evolução do código, os testes unitários demonstram com precisão executável como as classes, métodos e parâmetros devem ser chamados e quais comportamentos eles produzem. Como os testes são executados rotineiramente e falham quando o código muda sem conformidade, eles permanecem sempre sincronizados com o comportamento real do software.
