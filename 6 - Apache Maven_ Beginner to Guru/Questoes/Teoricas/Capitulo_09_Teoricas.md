# Questões Teóricas - Capítulo 09 (Testing with Maven)

**1. Surefire vs Failsafe:** Qual é a diferença fundamental de arquitetura e propósito entre o `maven-surefire-plugin` e o `maven-failsafe-plugin` no ciclo de vida do build?
> [!faq]- 👀 Ver Resposta
> * O **Surefire** é projetado para **testes unitários** isolados e rápidos; ele é executado na fase `test` e, se algum teste falhar, aborta imediatamente o restante do build com falha.
> * O **Failsafe** é projetado para **testes de integração** que interagem com bancos de dados ou servidores; ele roda na fase `integration-test` e, mesmo que um teste falhe, não interrompe o build imediatamente, permitindo que a fase subsequente `post-integration-test` desligue os servidores com segurança. O Failsafe só valida os resultados e falha o build na fase final `verify`.

**2. Convenções de Nomes para Testes de Integração:** Quais são os padrões de nomenclatura de arquivos que o `maven-failsafe-plugin` pesquisa por padrão para identificar classes de teste de integração?
> [!faq]- 👀 Ver Resposta
> O Failsafe pesquisa por padrão em `src/test/java` classes que sigam os seguintes sufixos ou prefixos:
> * `**/IT*.java` (classes iniciando com "IT" - Integration Test)
> * `**/*IT.java` (classes terminando com "IT")
> * `**/*ITCase.java` (classes terminando com "ITCase")
> Essa convenção impede que testes de integração pesados sejam executados acidentalmente pelo Surefire durante a fase rápida de testes unitários (`mvn test`).

**3. Testes POJO (Plain Old Java Objects):** O que é um teste no formato "POJO Test" historicamente suportado pelo Surefire e por que ele não exigia nenhuma anotação ou framework de teste importado?
> [!faq]- 👀 Ver Resposta
> Um teste POJO é uma classe Java comum que não herda de nenhuma biblioteca de testes (como JUnit) e não possui anotações como `@Test`. O Surefire antigo reconhecia métodos públicos cujo nome começava com `test` (ex: `public void testCalculo()`) e que lançavam exceções em caso de falha. Esse formato é um artefato histórico do início do Maven, hoje completamente em desuso frente a frameworks modernos.

**4. A Arquitetura do JUnit 5 (Platform, Jupiter e Vintage):** Explique resumidamente o papel de cada uma das três partes que compõem a arquitetura do JUnit 5 e como o `junit-vintage-engine` viabiliza a migração de bases de código legadas.
> [!faq]- 👀 Ver Resposta
> * **JUnit Platform**: É o motor base que serve como fundação para lançar frameworks de teste na JVM, além de fornecer a interface de integração para o Maven Surefire e IDEs.
> * **JUnit Jupiter**: É o novo modelo de programação e extensão para escrita de testes modernos em JUnit 5 (contendo anotações como `@Test`, `@BeforeEach`, `@ParameterizedTest`).
> * **JUnit Vintage**: É um mecanismo de compatibilidade que roda sobre a Platform, permitindo executar testes legados escritos em **JUnit 3 e JUnit 4** sem que seja necessário reescrever o código antigo.

**5. Execução do Spock Framework no Maven:** O que é o framework de testes **Spock** (baseado em Groovy) e qual motor do JUnit 5 a sua versão moderna (Spock 2.0+) utiliza para ser executada pelo `maven-surefire-plugin`?
> [!faq]- 👀 Ver Resposta
> O Spock é um framework de testes e especificações altamente expressivo escrito em Groovy, conhecido por sua sintaxe elegante baseada em blocos BDD (`given:`, `when:`, `then:`), data-driven testing com tabelas e mocking integrado. A partir do **Spock 2.0+**, ele foi totalmente reescrito para rodar como uma engine nativa sobre a **JUnit Platform**, permitindo que o Surefire o execute automaticamente sem configurações complexas de plugins de terceiros.

**6. Pular Testes de Forma Granular:** Qual é a diferença técnica entre passar o argumento `-DskipTests` e o argumento `-DskipITs` durante o comando `mvn clean verify`?
> [!faq]- 👀 Ver Resposta
> * `-DskipTests`: Sinalizador universal que instrui tanto o Surefire quanto o Failsafe a pularem a **execução** de todos os testes (unitários e de integração), embora as classes de teste continuem sendo compiladas em `target/test-classes`.
> * `-DskipITs`: Sinalizador específico do `maven-failsafe-plugin` que pula estritamente os **testes de integração**, permitindo que os testes unitários rápidos do Surefire continuem rodando normalmente durante o build.

**7. Relatórios de Teste com o `maven-surefire-report-plugin`:** O que o plugin de relatórios do Surefire produz durante o ciclo de build e em qual fase do ciclo de vida `site` ele geralmente opera?
> [!faq]- 👀 Ver Resposta
> Ele processa os arquivos de saída XML gerados pelo Surefire em `target/surefire-reports` e gera uma documentação visual rica em formato HTML contendo métricas completas: número total de testes executados, taxas de sucesso, testes ignorados, tempo exato de execução de cada teste e stacktraces detalhados de falhas, operando na fase de relatórios do ciclo de vida `site`.

**8. Cobertura de Código com o JaCoCo (`jacoco-maven-plugin`):** Como o JaCoCo mede a cobertura de testes de uma aplicação Java e quais são as duas metas fundamentais (`goals`) que devem ser vinculadas ao ciclo de vida do build?
> [!faq]- 👀 Ver Resposta
> O JaCoCo (*Java Code Coverage*) mede a cobertura através de instrumentação de bytecode em memória durante a execução dos testes via Java Agent. Seus dois goals fundamentais são:
> 1. `prepare-agent`: Executado antes dos testes para configurar a JVM e coletar dados de execução.
> 2. `report`: Executado após a suíte de testes para consolidar as métricas e gerar relatórios visuais (HTML/XML/CSV) indicando percentuais de linhas, instruções e ramos condicionais cobertos.

**9. Análise Estática de Código com o SpotBugs (`spotbugs-maven-plugin`):** Qual é a diferença conceitual entre o que um framework de testes (JUnit) avalia e o que uma ferramenta de análise estática como o SpotBugs avalia no projeto?
> [!faq]- 👀 Ver Resposta
> Os testes unitários (JUnit) avaliam a **corretude funcional** do código (se dada uma entrada X, o método produz a saída Y esperada). O SpotBugs, por sua vez, realiza uma **análise estática profunda do bytecode** sem rodar a aplicação, procurando por centenas de padrões comuns de erros arquiteturais e vulnerabilidades (*bug patterns*), como potenciais `NullPointerException`, vazamento de streams não fechados, falhas de concorrência e brechas de segurança.

**10. A Regra de Falha por Cobertura Mínima no JaCoCo:** Como pipelines de CI/CD corporativas utilizam o objetivo `jacoco:check` para garantir a qualidade contínua da base de código?
> [!faq]- 👀 Ver Resposta
> O objetivo `jacoco:check` permite definir regras rígidas de corte (*thresholds*) no `pom.xml` (por exemplo, exigir no mínimo 80% de cobertura de linhas ou ramos condicionais). Se uma nova funcionalidade for adicionada sem testes suficientes e a cobertura média cair abaixo do limite homologado, o plugin aborta o build do Maven com erro na esteira de CI/CD, impedindo que o código seja mesclado ou vá para produção sem testes adequados.
