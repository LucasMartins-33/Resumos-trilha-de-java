# Capítulo 08: Alternate JVM Languages

A Máquina Virtual Java (JVM) é uma plataforma poliglota capaz de executar qualquer linguagem cujos compiladores gerem bytecodes compatíveis com o padrão Java. Neste capítulo, é explorado como configurar o Apache Maven para compilar projetos híbridos contendo **Java + uma linguagem alternativa da JVM** (**Groovy**, **Kotlin** e **Scala**), empacotando os bytecodes gerados em um único arquivo `.jar` executável e garantindo a interoperabilidade bidirecional entre as linguagens.

---

## 1. O Desafio da Compilação Mista ("A Dança dos Compiladores")

Quando um projeto combina código Java com outra linguagem da JVM no mesmo módulo:
1. Uma classe escrita na linguagem alternativa (ex: Kotlin) precisa instanciar e referenciar tipos escritos em Java.
2. O código Java também pode precisar chamar métodos ou classes escritas nessa linguagem alternativa.

O compilador padrão do Maven (`maven-compiler-plugin`) não conhece nativamente linguagens externas. Portanto, os plugins específicos de cada linguagem precisam assumir o controle da fase `compile`, orquestrando a ordem de compilação ou substituindo as ações padrão.

```
src/main/
├── java/       ──> [ JavaHelloWorld.java ] ──┐
└── kotlin/     ──> [ Hello.kt ] ────────────┼──> [ kotlin-maven-plugin ] ──> target/classes/ ──> [ JAR Único ]
                                              │   (Gera bytecodes .class)
```

> [!TIP]
> **Regra de Ouro da Arquitetura:** Misture no máximo **duas linguagens** (Java + 1 alternativa) dentro do mesmo módulo Maven. Caso necessite de mais de uma linguagem alternativa, separe o projeto em módulos Maven distintos (*multi-module*).

---

## 2. Compilando Apache Groovy com Maven

Para compilar Groovy e Java conjuntamente, a abordagem demonstrada utiliza o **Groovy Eclipse Compiler** acoplado como provedor de compilação dentro do `maven-compiler-plugin`.

### 2.1. Configuração do `pom.xml`
```xml
<dependencies>
    <!-- Biblioteca runtime do Groovy -->
    <dependency>
        <groupId>org.codehaus.groovy</groupId>
        <artifactId>groovy-all</artifactId>
        <version>3.0.8</version>
        <type>pom</type>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.0</version>
            <configuration>
                <compilerId>groovy-eclipse-compiler</compilerId>
            </configuration>
            <dependencies>
                <!-- Dependências que ensinam o javac a compilar Groovy -->
                <dependency>
                    <groupId>org.codehaus.groovy</groupId>
                    <artifactId>groovy-eclipse-compiler</artifactId>
                    <version>3.7.0</version>
                </dependency>
                <dependency>
                    <groupId>org.codehaus.groovy</groupId>
                    <artifactId>groovy-eclipse-batch</artifactId>
                    <version>3.0.8-01</version>
                </dependency>
            </dependencies>
        </plugin>
    </plugins>
</build>
```

### 2.2. Estrutura e Interoperabilidade
* **Código Java:** `src/main/java/JavaHelloWorld.java`
  ```java
  public class JavaHelloWorld {
      public String getHello() {
          return "Hello World";
      }
  }
  ```
* **Código Groovy:** `src/main/groovy/GroovyHello.groovy`
  ```groovy
  class GroovyHello {
      static void main(String[] args) {
          def javaHello = new JavaHelloWorld()
          // No Groovy, getHello() pode ser acessado diretamente como propriedade
          println javaHello.hello 
      }
  }
  ```
* O compilador analisa ambas as árvores de código em um único passo, compilando ambas as classes para `target/classes/`.

---

## 3. Compilando Kotlin com Maven

A linguagem Kotlin (criada pela JetBrains) fornece suporte oficial de compilação mista através do plugin **`kotlin-maven-plugin`**.

### 3.1. Estratégia de Compilação
1. O plugin do Kotlin é configurado para executar primeiro na fase `compile`, analisando as fontes em `src/main/kotlin` e `src/main/java`.
2. A meta padrão `default-compile` do `maven-compiler-plugin` é desativada (`<phase>none</phase>`).
3. Uma nova execução do compilador Java é vinculada para finalizar as classes Java restantes.

### 3.2. Configuração do `pom.xml`
```xml
<properties>
    <kotlin.version>1.3.11</kotlin.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.jetbrains.kotlin</groupId>
        <artifactId>kotlin-stdlib</artifactId>
        <version>${kotlin.version}</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <!-- 1. Compilador Kotlin -->
        <plugin>
            <groupId>org.jetbrains.kotlin</groupId>
            <artifactId>kotlin-maven-plugin</artifactId>
            <version>${kotlin.version}</version>
            <executions>
                <execution>
                    <id>compile</id>
                    <goals> <goal>compile</goal> </goals>
                    <configuration>
                        <sourceDirs>
                            <sourceDir>${project.basedir}/src/main/kotlin</sourceDir>
                            <sourceDir>${project.basedir}/src/main/java</sourceDir>
                        </sourceDirs>
                    </configuration>
                </execution>
                <execution>
                    <id>test-compile</id>
                    <goals> <goal>test-compile</goal> </goals>
                    <configuration>
                        <sourceDirs>
                            <sourceDir>${project.basedir}/src/test/kotlin</sourceDir>
                            <sourceDir>${project.basedir}/src/test/java</sourceDir>
                        </sourceDirs>
                    </configuration>
                </execution>
            </executions>
        </plugin>

        <!-- 2. Compilador Java ajustado -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.0</version>
            <executions>
                <!-- Desativa a execução padrão para evitar compilar duas vezes -->
                <execution>
                    <id>default-compile</id>
                    <phase>none</phase>
                </execution>
                <execution>
                    <id>default-testCompile</id>
                    <phase>none</phase>
                </execution>
                <!-- Vincula a compilação Java após a compilação do Kotlin -->
                <execution>
                    <id>java-compile</id>
                    <phase>compile</phase>
                    <goals> <goal>compile</goal> </goals>
                </execution>
                <execution>
                    <id>java-test-compile</id>
                    <phase>test-compile</phase>
                    <goals> <goal>testCompile</goal> </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 3.3. Interoperabilidade em Ação
Arquivo `src/main/kotlin/Hello.kt`:
```kotlin
fun main(args: Array<String>) {
    val hi = JavaHelloWorld()
    println(hi.hello)
}
```
* Ao executar `mvn clean package`, o Maven compila tanto o código Java quanto o Kotlin e empacota ambos dentro do JAR final.

---

## 4. Compilando Scala com Maven e o Problema Histórico com Java 11

Para compilar Scala, utiliza-se o plugin **`scala-maven-plugin`** (`net.alchim31.maven`).

### 4.1. Configuração Típica
```xml
<dependencies>
    <dependency>
        <groupId>org.scala-lang</groupId>
        <artifactId>scala-library</artifactId>
        <version>2.11.7</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>net.alchim31.maven</groupId>
            <artifactId>scala-maven-plugin</artifactId>
            <version>3.4.4</version>
            <executions>
                <execution>
                    <id>scala-compile-first</id>
                    <phase>process-resources</phase>
                    <goals>
                        <goal>add-source</goal>
                        <goal>compile</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 4.2. O Erro Relatado na Aula (`MissingRequirementError` no Java 11)
Durante a gravação da aula (final de 2018), o Scala 2.11/2.12 ainda não era totalmente compatível com o sistema de módulos e com os bytecodes do **Java 11**, resultando em falhas de compilação:
* O compilador Scala da época dependia estritamente do JDK 8.
* A evolução dessa compatibilidade é detalhada no apêndice abaixo.

---

## 5. Apêndice — Atualizações & Boas Práticas Modernas

### 1. Fim do Bintray / JCenter e Centralização de Artefatos
Na aula sobre Groovy, o instrutor adiciona manualmente o repositório remoto do Bintray (`dl.bintray.com`) porque a versão do `groovy-eclipse-batch` não estava no Maven Central.
* **Fato Crítico:** A JFrog descontinuou e **encerrou permanentemente o Bintray/JCenter em 2021**. Qualquer build que ainda aponte para repositórios Bintray falhará com erro `403/404`.
* **Hoje:** O ecossistema Groovy migrou para a Apache Software Foundation (`org.apache.groovy`). Todas as versões modernas do Groovy (versões 3.x e 4.x), bem como seus plugins de compilação, estão publicadas diretamente no **Maven Central**, dispensando tags `<pluginRepositories>` adicionais.

### 2. GMavenPlus: A Alternativa Moderna para Groovy
Embora o Groovy-Eclipse continue ativo, a ferramenta mais adotada e recomendada pela comunidade Apache Groovy atualmente é o plugin **`gmavenplus-plugin`** (`org.codehaus.gmavenplus`):
```xml
<plugin>
    <groupId>org.codehaus.gmavenplus</groupId>
    <artifactId>gmavenplus-plugin</artifactId>
    <version>3.0.2</version>
    <executions>
        <execution>
            <goals>
                <goal>addSources</goal>
                <goal>addTestSources</goal>
                <goal>compile</goal>
                <goal>compileTests</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```
* O GMavenPlus integra-se de forma muito mais natural com testes escritos em **Spock Framework** e suporta versões modernas do Java (Java 17 e 21).

### 3. Resolução do Suporte a Java Moderno no Scala (Scala 2.13 e Scala 3)
A limitação que impedia a compilação do Scala no Java 11 foi completamente superada:
* **Scala 2.13** e **Scala 3** oferecem compatibilidade total e nativa com Java 11, Java 17 e Java 21 LTS.
* O plugin moderno `scala-maven-plugin` (versão `4.8.x` ou superior) compila perfeitamente em JDKs contemporâneos.

### 4. Panorama Contemporâneo das Linguagens Alternativas na JVM
* **Kotlin:** Tornou-se uma linguagem de primeira classe no backend enterprise, com suporte oficial de alto nível no **Spring Boot**, **Micronaut** e **Quarkus**, com coroutines nativas e APIs reativas.
* **Groovy:** Mantém posição de liderança inquestionável no ecossistema de testes corporativos através do framework **Spock** e em automações de CI/CD (Jenkins pipelines).
* **Scala:** Sofreu uma grande reformulação com o lançamento do **Scala 3**, mantendo forte presença no processamento de Big Data (Apache Spark, Kafka Streams e Akka/Apache Pekko).
