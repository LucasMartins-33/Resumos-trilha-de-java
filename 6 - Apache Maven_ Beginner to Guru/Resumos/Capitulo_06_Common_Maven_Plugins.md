# Capítulo 06: Common Maven Plugins

Neste capítulo são explorados os plugins essenciais que realizam o trabalho pesado no Apache Maven. O Maven é, essencialmente, um framework que orquestra a execução de plugins através de fases de ciclo de vida. São analisados os principais plugins nativos (`clean`, `compiler`, `resources`, `surefire`, `jar`, `deploy`, `site`), a anatomia de execução de metas dentro de fases, a criação de JARs executáveis, boas práticas de controle de versão (`.gitignore`) e os comandos mais utilizados no dia a dia.

---

## 1. O Conceito Central: Maven como Framework de Plugins

No Maven, **todo o trabalho real é executado por plugins**. As fases do ciclo de vida são apenas ganchos (*hooks*) de orquestração temporal.

* **Fase (*Phase*):** Uma etapa abstrata da esteira de compilação (ex: `compile`, `test`, `package`).
* **Meta do Plugin (*Plugin Goal*):** A função concreta que faz o trabalho (notação `plugin:goal`, ex: `compiler:compile`, `surefire:test`, `jar:jar`).

Podemos executar:
1. **Uma Fase:** `mvn compile` (O Maven executa todas as fases até `compile`, disparando todas as metas associadas).
2. **Um Plugin Goal Diretamente:** `mvn clean:clean` ou `mvn dependency:tree` (Executa isoladamente a meta do plugin sem passar pelo ciclo de vida).

---

## 2. Visão Geral dos Plugins Principais do Ciclo de Vida

| Plugin | Metas Principais (*Goals*) | Fase Típica Vinculada | Finalidade |
| :--- | :--- | :--- | :--- |
| **`maven-clean-plugin`** | `clean` | `clean` | Exclui os diretórios de saída (por padrão `target/`). |
| **`maven-compiler-plugin`** | `compile`, `testCompile` | `compile`, `test-compile` | Compila o código Java de produção e de teste em bytecodes. |
| **`maven-resources-plugin`** | `resources`, `testResources`, `copy-resources` | `process-resources`, `process-test-resources` | Copia arquivos de configuração e recursos estáticos para o classpath de saída. |
| **`maven-surefire-plugin`** | `test` | `test` | Executa a suíte de testes unitários e gera relatórios. |
| **`maven-jar-plugin`** | `jar`, `test-jar` | `package` | Empacota classes compiladas e recursos em um arquivo `.jar`. |
| **`maven-deploy-plugin`** | `deploy`, `deploy-file` | `deploy` | Publica o artefato gerado em um repositório remoto compartilhado. |
| **`maven-site-plugin`** | `site`, `deploy`, `run`, `stage` | `site` | Gera documentação HTML e o website do projeto. |

---

## 3. Detalhamento e Configuração dos Plugins

### 3.1. Maven Clean Plugin
Remove artefatos compilados de builds anteriores para evitar "lixo residual de refatoração" (classes renomeadas ou excluídas que continuariam no classpath se o `target/` não fosse limpo).

**Vinculando a limpeza automática ao ciclo de build (`initialize`):**
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-clean-plugin</artifactId>
            <version>3.1.0</version>
            <executions>
                <execution>
                    <id>auto-clean</id>
                    <phase>initialize</phase>
                    <goals>
                        <goal>clean</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
* Ao configurar isso, rodar simplesmente `mvn package` acionará a limpeza antes da compilação, eliminando a necessidade de digitar sempre `mvn clean package`.

---

### 3.2. Maven Compiler Plugin
Responsável pela compilação do código. Por padrão, utiliza a API `javax.tools` interna da JVM corrente.

* **Metas:** `compile` (para `src/main/java`) e `testCompile` (para `src/test/java`).
* **Configuração de versão Java:**
  ```xml
  <properties>
      <maven.compiler.source>11</maven.compiler.source>
      <maven.compiler.target>11</maven.compiler.target>
  </properties>
  ```

---

### 3.3. Maven Resources Plugin
Copia arquivos não-Java (como `.properties`, `.xml`, `.json`, imagens) para a pasta de classes compiladas (`target/classes`), permitindo que a aplicação os acerte via `ClassLoader`.

* Metas: `resources` (lê de `src/main/resources`), `testResources` (lê de `src/test/resources`) e `copy-resources` (cópia arbitrária).
* Suporta **Resource Filtering**: substituição dinâmica de variáveis `${propriedade}` em arquivos de configuração com valores definidos no POM ou perfis.

---

### 3.4. Maven Surefire Plugin
O executor padrão de testes unitários do Maven.

* Suporta múltiplos frameworks: JUnit 3, JUnit 4, JUnit 5 (Jupiter), TestNG, Spock e Cucumber.
* **Padrões de Nomenclatura Automáticos:** Por padrão, o Surefire executa qualquer classe que corresponda a:
  * `**/Test*.java`
  * `**/*Test.java`
  * `**/*Tests.java`
  * `**/*TestCase.java`
* **Relatórios:** Gera arquivos de resultado (em `.xml` e `.txt`) na pasta `target/surefire-reports/`, consumidos por ferramentas de CI (Jenkins, GitLab CI, GitHub Actions).
* **Testes POJO (*Plain Old Java Object*):** O Surefire também é capaz de executar classes comuns que contenham métodos iniciados por `test` retornando `void`, sem anotações (comportamento histórico).

---

### 3.5. Maven Jar Plugin e Criação de JAR Executável
Por padrão, o Maven gera um **Thin JAR** (contendo apenas as classes compiladas do projeto). Para torná-lo executável via `java -jar <arquivo>.jar`, configura-se o manifesto do JAR:

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <version>3.2.0</version>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <mainClass>guru.springframework.HelloWorld</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

#### Como Executar:
1. Compile e empacote:
   ```bash
   mvn package
   ```
2. O arquivo `target/META-INF/MANIFEST.MF` conterá as entradas:
   ```manifest
   Main-Class: guru.springframework.HelloWorld
   Class-Path: commons-lang3-3.8.1.jar
   ```
3. Para executar:
   ```bash
   java -jar target/hello-world-1.0-SNAPSHOT.jar
   ```
   > **Atenção:** Em um Thin JAR com dependências externas, o comando `java -jar` espera que os JARs dependentes (ex: `commons-lang3.jar`) estejam presentes no mesmo diretório em que o comando está sendo executado ou no caminho configurado no classpath do manifesto.

---

### 3.6. Maven Deploy Plugin
Publica o artefato construído em um repositório remoto (ex: Nexus, Artifactory ou Maven Central).

* Não é configurado com parâmetros soltos na tag do plugin, mas sim na tag padronizada do POM `<distributionManagement>`:
  ```xml
  <distributionManagement>
      <repository>
          <id>meu-nexus-releases</id>
          <url>https://nexus.minhaempresa.com/repository/maven-releases/</url>
      </repository>
      <snapshotRepository>
          <id>meu-nexus-snapshots</id>
          <url>https://nexus.minhaempresa.com/repository/maven-snapshots/</url>
      </snapshotRepository>
  </distributionManagement>
  ```
* As credenciais sensíveis (usuário e senha) **nunca ficam no `pom.xml`**; elas são declaradas no arquivo local do usuário `~/.m2/settings.xml` vinculadas pelo `<id>`.

---

### 3.7. Maven Site Plugin
Gera relatórios completos e documentação do projeto em formato de website estático (HTML/CSS), lendo dados estruturados do POM (desenvolvedores, repositório de código, dependências e relatórios de plugins).

* Formatos de documentação suportados: Markdown, APT (*Almost Plain Text*), XDoc e HTML.
* Comando para gerar: `mvn site` (gera o site dentro de `target/site/index.html`).

---

## 4. Boas Práticas: Maven e Controle de Versão (Git)

Um dos erros mais comuns de iniciantes é commitar arquivos gerados temporariamente ou metadados de IDEs locais.

### O que DEVE ser versionado no Git:
* `pom.xml`
* Pasta `src/` (`src/main`, `src/test`)
* Scripts do Maven Wrapper: `mvnw` e `mvnw.cmd`
* Pasta oculta do Maven Wrapper: `.mvn/wrapper/` (incluindo `maven-wrapper.properties` e `maven-wrapper.jar`)
* Arquivo `.gitignore`

### O que NUNCA deve ser commitado:
* **Pasta `target/`:** Todo o conteúdo gerado em tempo de compilação.
* **Arquivos específicos de IDE:**
  * IntelliJ: pasta `.idea/`, arquivos `*.iml`.
  * Eclipse: arquivos `.project`, `.classpath`, pasta `.settings/`.
  * NetBeans: pasta `nbproject/`.
* **Arquivos de Sistema Operacional:** `.DS_Store` (macOS), `Thumbs.db` (Windows).

---

## 5. Tabela Rápida de Comandos Maven (*CheatSheet*)

| Comando | Descrição |
| :--- | :--- |
| `mvn clean` | Limpa a pasta `target/`. |
| `mvn compile` | Compila o código-fonte de produção. |
| `mvn test` | Compila e executa os testes unitários. |
| `mvn package` | Compila, testa e gera o pacote (`.jar` ou `.war`). |
| `mvn install` | Executa o `package` e copia o artefato gerado para o repositório local (`~/.m2/repository`). |
| `mvn deploy` | Envia o artefato para o repositório remoto configurado. |
| `mvn clean install` | Limpa o ambiente e executa o ciclo completo até a instalação local. |
| `mvn test -Dtest=HelloWorldTest` | Executa apenas a classe de teste especificada. |
| `mvn package -DskipTests` | Compila os testes, mas não os executa durante o empacotamento. |
| `mvn dependency:tree` | Imprime no console a árvore hierárquica completa de dependências. |
| `mvn help:effective-pom` | Exibe o POM consolidado com todas as heranças e valores padrão do Maven. |

---

## 6. Apêndice — Atualizações & Boas Práticas Modernas

### 1. Propriedades Modernas do Compilador: `<maven.compiler.release>`
Em vez de especificar separadamente `maven.compiler.source` e `maven.compiler.target`, utilize a diretiva única para JDKs modernos:
```xml
<properties>
    <maven.compiler.release>17</maven.compiler.release>
</properties>
```

### 2. Testes Modernos: JUnit 5 (Jupiter) vs Testes POJO
* Testes no formato POJO (sem anotações, baseados no nome do método `test...`) são uma convenção histórica obsoleta.
* A prática contemporânea adota exclusivamente o **JUnit 5 (Jupiter)** com a anotação `@Test` e métodos com visibilidade de pacote (não precisam ser `public`):
  ```java
  import org.junit.jupiter.api.Test;
  import static org.junit.jupiter.api.Assertions.assertEquals;

  class HelloWorldTest {
      @Test
      void shouldCapitalizeString() {
          assertEquals("Hello world", StringUtils.capitalize("hello world"));
      }
  }
  ```

### 3. Gerando JARs Executáveis Completos (Fat JAR / Uber-JAR)
O método abordado na aula com `maven-jar-plugin` gera um *Thin JAR* (que depende dos JARs externos estarem presentes na mesma pasta). No ecossistema moderno, utiliza-se uma das seguintes abordagens para gerar um **Fat JAR único e auto-contido**:
1. **Spring Boot Plugin:**
   ```xml
   <plugin>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-maven-plugin</artifactId>
   </plugin>
   ```
2. **Maven Shade Plugin (`maven-shade-plugin`):**
   Descompacta todas as classes de todas as dependências e as junta em um único super JAR executável, ideal para aplicações CLI e microsserviços sem Spring.

### 4. Gerenciamento Centralizado de Versões de Plugins (`<pluginManagement>`)
No vídeo, omitir a versão de plugins causou avisos de inconsistência. Em projetos reais e multi-módulos, as versões dos plugins devem ser travadas em um bloco `<pluginManagement>` no POM pai ou herdadas de um Parent homologado, garantindo builds reproduzíveis em qualquer máquina ou pipeline de CI.

### 5. Substituição do Maven Site na Indústria
Embora o `maven-site-plugin` ainda exista e mantenha a documentação oficial dos plugins do Apache Maven, a maioria das empresas e projetos modernos migrou a geração de sites e documentação para ferramentas especializadas como **MkDocs**, **Docusaurus**, **Antora** ou geração de especificações de API via **OpenAPI / Swagger** (`springdoc-openapi`).
