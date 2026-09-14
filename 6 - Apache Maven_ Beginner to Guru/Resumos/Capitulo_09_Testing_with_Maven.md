# Capítulo 09: Testing with Maven

Neste capítulo é explorada a infraestrutura completa de **testes automatizados e garantia de qualidade** com o Apache Maven. O Maven possui separação rígida e conceitual entre **testes unitários** (executados pelo `maven-surefire-plugin` na fase `test`) e **testes de integração** (executados pelo `maven-failsafe-plugin` na fase `integration-test`/`verify`). Além dos executores, são analisados frameworks de testes (POJO, JUnit 4, JUnit 5, TestNG, Spock), geração de relatórios gráficos, análise de cobertura de código com **JaCoCo**, análise estática de vulnerabilidades com **SpotBugs** e técnicas para pular testes seletivamente.

---

## 1. Arquitetura de Testes no Maven: Surefire vs Failsafe

O Maven separa testes em duas categorias com objetivos, tempos de execução e fases de ciclo de vida distintos:

```
[ validate ] ──> [ compile ] ──> [ test-compile ] ──> [ TEST ] ──(Surefire: Testes Unitários Rápidos)
                                                         │
                                                  (Se passar)
                                                         ▼
[ package (gera JAR/WAR) ] ──> [ pre-integration-test ] ──> [ INTEGRATION-TEST ] ──(Failsafe: Testes Pesados)
                                                                    │
                                                           [ post-integration-test ] ──(Teardown / Limpeza)
                                                                    │
                                                               [ VERIFY ] ──(Failsafe: Valida se houve falhas)
```

### 1.1. Comparativo Surefire vs Failsafe

| Característica | `maven-surefire-plugin` (Testes Unitários) | `maven-failsafe-plugin` (Testes de Integração) |
| :--- | :--- | :--- |
| **Fase do Ciclo de Vida** | `test` | `integration-test` e `verify` |
| **Comando de Execução** | `mvn test` | `mvn verify` |
| **Padrão de Nomenclatura** | `**/Test*.java`, `**/*Test.java`, `**/*TestCase.java` | `**/IT*.java`, `**/*IT.java`, `**/*ITCase.java` |
| **Configuração no POM** | Embutido por padrão no Super POM do Maven. | Requer configuração explícita de `<executions>` no `pom.xml`. |
| **Comportamento em Falhas** | Aborta o build **imediatamente** no primeiro erro. | Executa todos os testes até o final e só aborta na fase `verify`, garantindo que contêineres e bancos de teste possam ser destruídos na fase `post-integration-test`. |

---

## 2. Frameworks de Testes Suportados

### 2.1. Testes POJO (*Plain Old Java Object*)
* **Conceito:** O Surefire consegue executar testes sem depender de nenhum framework externo.
* **Regras:**
  * O nome da classe deve terminar com `Test` (ex: `JavaHelloWorldTest.java`).
  * O método de teste deve começar com `test` e retornar `void` (ex: `public void testHello()`).
  * As validações utilizam a palavra-chave nativa `assert` do Java (`assert "Hello World".equals(result);`).

---

### 2.2. JUnit 4
Padrão histórico consagrado da indústria:

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```
* **Uso no código:** Anotação `@Test` de `org.junit.Test` e métodos utilitários estáticos como `org.junit.Assert.assertEquals(...)`.

---

### 2.3. JUnit 5 (Jupiter)
Framework moderno e modular do ecossistema Java.

#### Configuração das Dependências no `pom.xml`:
```xml
<properties>
    <junit-jupiter.version>5.10.2</junit-jupiter.version>
</properties>

<dependencies>
    <!-- API com anotações e asserções -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>${junit-jupiter.version}</version>
        <scope>test</scope>
    </dependency>
    <!-- Engine que executa os testes Jupiter no Surefire -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>${junit-jupiter.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- Requer Surefire 2.22.0 ou superior para suporte nativo ao JUnit 5 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.5</version>
        </plugin>
    </plugins>
</build>
```

#### Uso no Código Java:
```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

class JavaHelloWorldTest {
    @Test
    void testHello() {
        JavaHelloWorld hello = new JavaHelloWorld();
        assertEquals("Hello World", hello.getHello());
    }
}
```

---

### 2.4. Coexistência de JUnit 4 e JUnit 5 no Mesmo Projeto
Para permitir que testes legados (JUnit 4) e novos testes (JUnit 5) rodem juntos na mesma esteira do Maven, adiciona-se o **JUnit Vintage Engine**:

```xml
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>${junit-jupiter.version}</version>
    <scope>test</scope>
</dependency>
```

---

### 2.5. TestNG
Framework de testes focado em testes de integração e configurações avançadas de grupos e paralelismo:

```xml
<dependency>
    <groupId>org.testng</groupId>
    <artifactId>testng</artifactId>
    <version>7.9.0</version>
    <scope>test</scope>
</dependency>
```
* Anotação: `@org.testng.annotations.Test`.
* É suportado nativamente pelo Surefire sem necessidade de plugins extras.

---

### 2.6. Spock Framework (BDD com Groovy)
Framework BDD baseado em especificações expressivas em Groovy (`given:`, `when:`, `then:`):

* Dependência: `org.spockframework:spock-core`.
* Convenção de pasta: `src/test/groovy/`.
* Nomenclatura de classe: `*Test.groovy` ou `*Spec.groovy` configurado nas inclusões do Surefire.

---

## 3. Configurando Testes de Integração com o Maven Failsafe Plugin

Para habilitar a execução de testes de integração na fase `verify`:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-failsafe-plugin</artifactId>
            <version>3.2.5</version>
            <executions>
                <execution>
                    <goals>
                        <goal>integration-test</goal>
                        <goal>verify</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

* **Convenção de Nomenclatura:** Crie a classe de teste como `JavaHelloWorldIT.java` dentro de `src/test/java/`.
* **Executando:**
  ```bash
  mvn verify
  ```
  *(O Maven executará primeiro os testes unitários com Surefire e, em seguida, os testes de integração com Failsafe)*.

---

## 4. Cobertura de Código com JaCoCo (*Java Code Coverage*)

O **JaCoCo** mede a porcentagem de linhas, branches e instruções executadas pelos testes.

### 4.1. Configuração do `pom.xml`
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.jacoco</groupId>
            <artifactId>jacoco-maven-plugin</artifactId>
            <version>0.8.11</version>
            <executions>
                <!-- 1. Prepara o Java Agent antes dos testes rodarem -->
                <execution>
                    <id>prepare-agent</id>
                    <goals>
                        <goal>prepare-agent</goal>
                    </goals>
                </execution>
                <!-- 2. Gera o relatório HTML de cobertura após a fase test/verify -->
                <execution>
                    <id>report</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>report</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
* **Resultado:** O relatório HTML interativo é gerado em `target/site/jacoco/index.html`.

---

## 5. Análise Estática de Código com SpotBugs

O **SpotBugs** é o sucessor oficial e mantido do clássico *FindBugs*. Ele escaneia os bytecodes compilados procurando por vulnerabilidades de segurança, vazamentos de memória e más práticas de codificação.

```xml
<reporting>
    <plugins>
        <plugin>
            <groupId>com.github.spotbugs</groupId>
            <artifactId>spotbugs-maven-plugin</artifactId>
            <version>4.8.3.1</version>
        </plugin>
    </plugins>
</reporting>
```
* Executado via `mvn site`, gerando o relatório na documentação estática do projeto.

---

## 6. Pulando Testes no Maven (*Skipping Tests*)

Em ambientes de CI/CD ou durante refatorações locais urgentes, pode ser necessário empacotar o projeto sem executar a bateria de testes.

| Cenário | Comando via Linha de Comando | O que faz |
| :--- | :--- | :--- |
| **Pular execução de TODOS os testes** *(Recomendado)* | `mvn package -DskipTests` | Compila as classes de teste (garantindo que o código de teste compila sem erros), mas **não as executa**. |
| **Pular compilação e execução de TODOS os testes** | `mvn package -Dmaven.test.skip=true` | Pula tanto a compilação quanto a execução de qualquer código de teste. |
| **Pular APENAS testes de integração** | `mvn verify -DskipITs` | Executa os testes unitários do Surefire normalmente, mas **ignora os testes pesados de integração do Failsafe**. |
| **Executar apenas um teste unitário específico** | `mvn test -Dtest=JavaHelloWorldTest` | Roda exclusivamente a classe indicada. |
| **Executar apenas um método específico** | `mvn test -Dtest=JavaHelloWorldTest#testHello` | Roda apenas o método de teste indicado dentro daquela classe. |

---

## 7. Apêndice — Atualizações & Boas Práticas Modernas

### 1. Dependência Unificada do JUnit 5: `junit-jupiter`
No vídeo, o instrutor adicionou separadamente `junit-jupiter-api` e `junit-jupiter-engine`.
* **Hoje:** O JUnit 5 disponibiliza o artefato agregador oficial **`junit-jupiter`**, que inclui a API, a Engine e o módulo de testes parametrizados (`junit-jupiter-params`) em uma única linha:
  ```xml
  <dependency>
      <groupId>org.junit.jupiter</groupId>
      <artifactId>junit-jupiter</artifactId>
      <version>5.10.2</version>
      <scope>test</scope>
  </dependency>
  ```

### 2. Spock 2.0+ e o Fim da Dependência do JUnit 4
Na aula, o instrutor explicou que o Spock 1.x compilava para bytecodes do JUnit 4 e, portanto, necessitava do `junit-vintage-engine` para rodar sob o JUnit 5.
* **Hoje:** O **Spock 2.0+** foi completamente reescrito para rodar diretamente sobre a **JUnit Platform** (`junit-platform-engine`). Ele **não precisa mais do JUnit 4 nem do Vintage Engine**; o Surefire moderno executa Spock 2.x nativamente pela JUnit Platform.

### 3. Surefire / Failsafe Versão 3.x+ Estável
Durante a gravação original do curso, o Maven Surefire estava na versão experimental `3.0.0-M2` com bugs de detecção fantasma de TestNG e avisos de reflexão no Java 11.
* Hoje, as versões estáveis **3.2.x / 3.5.x** resolvem esses conflitos de compatibilidade, possuem suporte integral ao Java 17 e 21 LTS e suportam paralelismo nativo configurável (`<parallel>methods</parallel>`, `<threadCount>4</threadCount>`).

### 4. JaCoCo e o Padrão Moderno `@{argLine}`
Em versões modernas do Surefire e JaCoCo, a injeção do agente JVM é feita de forma totalmente transparente através da propriedade de expansão tardia `@{argLine}`. Isso evita o problema comum em versões antigas em que argumentos de memória ou JVM eram sobrescritos acidentalmente pelo JaCoCo.
