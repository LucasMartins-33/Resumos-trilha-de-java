# Capítulo 11: Apache Maven for Spring Boot

Este capítulo explora a profunda e engenhosa integração entre o **Apache Maven** e o **Spring Boot**. O Spring Boot revolucionou o desenvolvimento corporativo Java eliminando configurações manuais repetitivas, e grande parte dessa "mágica" é orquestrada diretamente pelas convenções, BOMs (*Bill of Materials*), gerenciamento de dependências e plugins construídos sobre o Maven.

---

## 1. Conceitos Fundamentais

### 1.1 Hierarquia de Herança: `spring-boot-starter-parent`
Ao criar uma aplicação Spring Boot baseada em Maven padrão, o projeto define uma relação de herança direta com o starter parent:
```text
Projeto do Desenvolvedor (pom.xml)
  └── spring-boot-starter-parent
        └── spring-boot-dependencies (BOM - Bill of Materials)
              └── spring-boot-build (configurações base da Pivotal/Broadcom)
                    └── Super POM do Maven
```

* **`spring-boot-dependencies`**: Atua como o catálogo central de curadoria de versões (`<dependencyManagement>`). Define centenas de bibliotecas compatíveis e testadas em conjunto (Spring, Jackson, Hibernate, Kafka, ActiveMQ, JUnit, Flyway, etc.), além de `<pluginManagement>` com parâmetros prontos para compilação, testes e empacotamento.
* **`spring-boot-starter-parent`**: Herda o catálogo do BOM e aplica configurações práticas de build:
  * Nível de compilação padrão (Java version).
  * Codificação de caracteres UTF-8 (`project.build.sourceEncoding`).
  * Filtragem de recursos (*resource filtering*) inteligente: filtra arquivos `application.properties` e `application.yml`, suportando placeholders delimitados por `@...@@` (ex: `@project.version@`), evitando colisão com sintaxes `${...}` do Spring.
  * Configurações padrão de execução do `maven-failsafe-plugin`, `maven-jar-plugin` e `maven-war-plugin`.

### 1.2 Sobrescrita de Propriedades (*Properties Override*)
Como o `spring-boot-dependencies` organiza as versões de dezenas de bibliotecas em tags `<properties>`, você não precisa alterar dependências manualmente para trocar a versão de um driver ou componente; basta redeclarar a propriedade no `<properties>` do seu `pom.xml`:
* Exemplo: mudar versão do Java (`<java.version>17</java.version>`), do Hibernate, Kafka, etc.

### 1.3 Spring Boot Starters
Starters são dependências agregadoras de conveniência que encapsulam um ecossistema completo de bibliotecas para um propósito específico (ex: `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `spring-boot-starter-test`):
* **Sem versão explícita**: Como o parent já define a versão compatível, omite-se a tag `<version>`.
* **Zero Boilerplate**: Elimina a necessidade de incluir manualmente `spring-web`, `spring-webmvc`, `jackson-databind`, `tomcat-embed-core`, etc.

### 1.4 Anatomia do Fat JAR / Executable JAR
O `spring-boot-maven-plugin` introduz o objetivo `repackage`, que transforma o arquivo JAR comum gerado pelo compilador em um **Fat JAR** (ou Uber JAR executável autossuficiente):

```text
meu-app.jar
├── META-INF/
│   ├── MANIFEST.MF
│   └── maven/
├── BOOT-INF/
│   ├── classes/               <-- Bytecode (.class) e resources do seu app
│   │   ├── com/meuapp/...
│   │   └── application.yml
│   └── lib/                   <-- Dependências externas embutidas (.jar aninhados)
│       ├── spring-boot-*.jar
│       ├── tomcat-embed-*.jar
│       └── jackson-*.jar
└── org/springframework/boot/loader/  <-- Classes do Boot Loader descompactadas
    ├── JarLauncher.class
    └── LaunchedURLClassLoader.class
```

#### Por que não usar o `maven-shade-plugin` simples?
Diferente do shade plugin (que descompacta e mescla todos os `.class` de todos os jars gerados no mesmo diretório raiz, podendo causar colisões de nomes e sobrescrita acidental de recursos/licenças), o Spring Boot mantém os JARs de dependência intactos dentro de `BOOT-INF/lib/`. 
* O `MANIFEST.MF` aponta o `Main-Class` para o carregador especial `org.springframework.boot.loader.JarLauncher`.
* A sua classe com o método `main` real fica registrada sob o atributo `Start-Class`.
* O `JarLauncher` inicializa um `LaunchedURLClassLoader` capaz de carregar classes a partir de JARs aninhados dentro de outro JAR.

### 1.5 Ciclo de Execução e Testes de Integração
* **`mvn spring-boot:run`**: Executa a aplicação diretamente no terminal a partir do código compilado (foreground).
* **`spring-boot:start` e `spring-boot:stop`**: O plugin fornece goals capazes de iniciar a aplicação em background antes dos testes de integração (`pre-integration-test`) e encerrá-la após o término (`post-integration-test`), integrando-se nativamente ao `maven-failsafe-plugin`.

### 1.6 Metadados de Build e Git (Actuator)
* **`build-info`**: Gera o arquivo `META-INF/build-info.properties` com dados da compilação (tempo, versão, artefato).
* **`git-commit-id-plugin`**: Inspeciona o repositório Git local e injeta um `git.properties` com branch, commit hash, commit message e tags.
* Ambos os arquivos são consumidos automaticamente pelo **Spring Boot Actuator** e expostos no endpoint `/actuator/info`.

### 1.7 Multi-Module Projects e o "Repackage Trap"
Em projetos multimódulos Spring Boot, é comum termos módulos de bibliotecas compartilhadas (ex: `model`, `services`, `domain`) e um módulo executável (ex: `web-app`).
* **O Problema**: Se o `spring-boot-maven-plugin` estiver ativo na raiz ou herdado indistintamente, o goal `repackage` tentará empacotar as bibliotecas de domínio como Fat JARs. Como módulos de biblioteca não possuem método `main`, o build falha com o erro:
  `Execution repackage of goal org.springframework.boot:spring-boot-maven-plugin failed: Unable to find main class`.
* Além disso, se um módulo for transformado em Fat JAR, ele não pode ser consumido como dependência normal por outro módulo (`BOOT-INF/classes` não fica visível no classpath tradicional).
* **A Solução**: Desativar o repackage nos módulos utilitários/biblioteca usando a propriedade `<spring-boot.repackage.skip>true</spring-boot.repackage.skip>` ou configurando `<skip>true</skip>` no plugin.

---

## 2. Sintaxe, Comandos & Configurações

### 2.1 Herança vs. Importação via BOM (`<dependencyManagement>`)

#### Opção Padrão: Herança com `spring-boot-starter-parent`
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.4</version>
    <relativePath/> <!-- busca diretamente do repositório remoto/cache local -->
</parent>
```

#### Opção Corporativa (Sem herança - Quando já existe um Parent empresarial)
Se a sua empresa exige um parent POM customizado (`corp-parent-pom`), use o escopo `import`:
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.2.4</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 2.2 Sobrescrita de Propriedades no `pom.xml`
```xml
<properties>
    <java.version>17</java.version>
    <!-- Sobrescrevendo a versão de dependências gerenciadas pelo BOM -->
    <activemq.version>5.18.3</activemq.version>
    <h2.version>2.2.224</h2.version>
</properties>
```

### 2.3 Configuração do `spring-boot-maven-plugin` com Build Info e Testes
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <executions>
                <!-- Gera META-INF/build-info.properties para o Actuator -->
                <execution>
                    <id>build-info</id>
                    <goals>
                        <goal>build-info</goal>
                    </goals>
                </execution>
                <!-- Sobe a aplicação antes dos testes de integração -->
                <execution>
                    <id>pre-it</id>
                    <phase>pre-integration-test</phase>
                    <goals>
                        <goal>start</goal>
                    </goals>
                </execution>
                <!-- Finaliza a aplicação após os testes de integração -->
                <execution>
                    <id>post-it</id>
                    <phase>post-integration-test</phase>
                    <goals>
                        <goal>stop</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 2.4 Integração com Git Metadata (`git-commit-id-maven-plugin`)
```xml
<plugin>
    <groupId>io.github.git-commit-id</groupId>
    <artifactId>git-commit-id-maven-plugin</artifactId>
    <executions>
        <execution>
            <id>get-the-git-infos</id>
            <goals>
                <goal>revision</goal>
            </goals>
            <phase>initialize</phase>
        </execution>
    </executions>
    <configuration>
        <generateGitPropertiesFile>true</generateGitPropertiesFile>
        <generateGitPropertiesFilename>${project.build.outputDirectory}/git.properties</generateGitPropertiesFilename>
        <includeOnlyProperties>
            <includeOnlyProperty>^git.branch$</includeOnlyProperty>
            <includeOnlyProperty>^git.build.(host|time|user)$</includeOnlyProperty>
            <includeOnlyProperty>^git.commit.id.(full|abbrev)$</includeOnlyProperty>
            <includeOnlyProperty>^git.commit.message.short$</includeOnlyProperty>
            <includeOnlyProperty>^git.commit.time$</includeOnlyProperty>
        </includeOnlyProperties>
        <commitIdGenerationMode>full</commitIdGenerationMode>
    </configuration>
</plugin>
```

### 2.5 Configuração em Módulos Compartilhados (Biblioteca / Model)
No módulo que **não** deve ser empacotado como Fat JAR:
```xml
<properties>
    <!-- Impede o repackage em módulos utilitários compartilhados -->
    <spring-boot.repackage.skip>true</spring-boot.repackage.skip>
</properties>
```
*Ou, se configurado via plugin execution:*
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <skip>true</skip>
    </configuration>
</plugin>
```

---

## 3. Comandos Maven para Spring Boot

| Comando | Descrição |
| :--- | :--- |
| `mvn spring-boot:run` | Inicia a aplicação no terminal local. |
| `mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8081"` | Passa argumentos de linha de comando para o Spring Boot. |
| `mvn spring-boot:run -Dspring-boot.run.profiles=dev,debug` | Inicia a aplicação ativando profiles específicos do Spring. |
| `mvn clean package` | Compila, testa e gera o Fat JAR executável na pasta `target/`. |
| `java -jar target/meu-app-0.0.1-SNAPSHOT.jar` | Executa diretamente o Fat JAR compilado via JVM. |
| `mvn verify` | Compila, sobe o app (`start`), roda os testes de integração (`Failsafe`) e desliga o app (`stop`). |
| `mvn spring-boot:build-image` | *(Moderno)* Cria uma imagem de container Docker/OCI usando Cloud Native Buildpacks sem precisar de `Dockerfile`. |

---

## 4. Apêndice — Atualizações & Boas Práticas Modernas

### 4.1 Linha de Base: Spring Boot 3.x
* **Java 17+ Obrigatório**: O Spring Boot 3.0+ abandonou o suporte ao Java 8 e 11. O baseline mínimo de runtime é Java 17 LTS, com suporte integral a Java 21 LTS e recursos modernos (Virtual Threads, Records, Pattern Matching).
* **Transição Jakarta EE 10**: O namespace `javax.*` foi completamente substituído por `jakarta.*` em dependências como Servlets, JPA/Hibernate, Bean Validation e JAXB:
  * De: `javax.persistence.*` ➔ Para: `jakarta.persistence.*`
  * De: `javax.validation.*` ➔ Para: `jakarta.validation.*`

### 4.2 Imagens OCI Nativas sem Dockerfile (`build-image`)
Desde o Spring Boot 2.3+, o plugin oficial inclui suporte nativo a **Cloud Native Buildpacks (Paketo)**:
```bash
mvn spring-boot:build-image
```
* Gera imagens de contêiner OCI altamente otimizadas, em camadas (*layered jars*), sem precisar criar ou manter um arquivo `Dockerfile`.
* Otimiza cache de CI/CD separando dependências estáticas do código de negócio (`classes`).

### 4.3 Compilação Nativa AOT com GraalVM
No Spring Boot 3.x, a compilação nativa com **GraalVM Native Image** tornou-se recurso de primeira classe:
```bash
mvn -Pnative native:compile
```
* Realiza análise estática antecipada (*Ahead-of-Time - AOT*).
* Produz um executável binário de máquina autônomo (não necessita de JVM instalada).
* Reduz o tempo de inicialização de segundos para milissegundos (~0.05s) e consome uma fração ínfima de memória RAM, ideal para ambientes Serverless e Kubernetes.

### 4.4 Migração do `git-commit-id-plugin`
A versão antiga mantida sob `pl.project13.maven:git-commit-id-plugin` foi descontinuada e agora reside oficialmente sob:
```xml
<groupId>io.github.git-commit-id</groupId>
<artifactId>git-commit-id-maven-plugin</artifactId>
```
Com compatibilidade total com os plugins e autoconfigurações do Spring Boot 3.x.

### 4.5 Boas Práticas para Estrutura Multimódulo com Spring Boot
1. **Mantenha o `spring-boot-starter-parent` apenas no módulo root**: Se o projeto for 100% Spring Boot, o parent root herda dele; caso contrário, use o padrão de BOM import via `<dependencyManagement>`.
2. **Separe Módulos Executáveis de Bibliotecas**:
   * Módulos de domínio (`domain`, `common`, `api-client`): devem gerar JARs regulares (repackage desativado).
   * Módulos executáveis (`server`, `web-api`): contêm o `@SpringBootApplication`, a classe `main` e ativam o `spring-boot-maven-plugin` para gerar o Fat JAR ou a Imagem de Container.
