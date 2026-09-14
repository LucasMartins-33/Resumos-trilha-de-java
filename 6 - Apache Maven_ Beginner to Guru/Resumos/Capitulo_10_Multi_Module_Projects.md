# Capítulo 10: Multi-Module Projects

Neste capítulo é explorada a arquitetura de **Projetos Multi-Módulos** (*Multi-Module Projects*) no Apache Maven. Projetos corporativos de médio e grande porte raramente consistem em um único arquivo JAR/WAR monolítico; em vez disso, são divididos em submódulos coesos (entidades de domínio, bibliotecas de conversão, APIs, interfaces web). São examinados o **Maven Reactor**, a gestão centralizada com **Parent POM**, o desacoplamento de versões dinâmicas via **`${revision}`**, a higienização de artefatos com o **Flatten Plugin**, a validação de regras com o **Enforcer Plugin** e a padronização corporativa via **Bill of Materials (BOM)**.

---

## 1. Arquitetura Multi-Módulo e o Maven Reactor

Um projeto multi-módulo é composto por um **POM Raiz (Parent/Agregador)** que orquestra submódulos através da tag `<modules>`.

```
meu-projeto/
├── pom.xml                     # Parent POM (packaging: pom) - Agregador & Gestor
├── jpa-entities/               # Submódulo de persistência (packaging: jar)
│   └── pom.xml
├── web-api/                    # Submódulo de contratos/DTOs (packaging: jar)
│   └── pom.xml
├── converters/                 # Submódulo de mapeamentos MapStruct (packaging: jar)
│   └── pom.xml
└── web-app/                    # Aplicação web final / Controllers (packaging: jar/war)
    └── pom.xml
```

### 1.1. O que é o Maven Reactor?
O **Reactor** é o motor de orquestração do Maven responsável por:
1. Inspecionar todos os submódulos declarados no bloco `<modules>`.
2. Analisar o grafo direcionado acíclico (DAG) de dependências entre os módulos.
3. Determinar a **Ordem de Compilação do Reactor (*Reactor Build Order*)**:
   * O Parent POM é processado primeiro.
   * Módulos sem dependências internas compilam em seguida.
   * Módulos dependentes de outros submódulos compilam após seus pré-requisitos estarem prontos.
4. Executar as fases do ciclo de vida em cada módulo sequencialmente.

```
Reactor Build Order:
1. mb2g-mm-maven (Parent / Agregador)
2. jpa-entities  (Independente)
3. web-api       (Independente)
4. converters    (Depende de jpa-entities e web-api)
5. web-app       (Depende de converters, jpa-entities e web-api)
```

### 1.2. Antipatronagem de Design (*Code Smells* na Modularização)
> [!WARNING]
> **Modularização Excessiva:** Criar módulos para uma única classe ou apenas para conter interfaces ("caso alguém precise no futuro") é um forte *code smell*. Cada módulo adicional faz o Reactor inicializar o ciclo de vida completo de plugins, multiplicando o tempo total de build. Divida módulos por **fronteiras de responsabilidade reais**, nunca por adivinhação de necessidades futuras.

---

## 2. Estrutura do Parent POM e dos Submódulos

### 2.1. O Parent POM (`pom.xml` da raiz)
O projeto agregador principal deve possuir `<packaging>pom</packaging>` e listar os submódulos:

```xml
<groupId>guru.springframework</groupId>
<artifactId>mb2g-mm-maven</artifactId>
<version>${revision}</version>
<packaging>pom</packaging>

<properties>
    <revision>1.0-SNAPSHOT</revision>
    <java.version>11</java.version>
    <maven.compiler.release>11</maven.compiler.release>
</properties>

<modules>
    <module>jpa-entities</module>
    <module>web-api</module>
    <module>converters</module>
    <module>web-app</module>
</modules>
```
* O diretório do Parent POM não possui pasta `src/`; ele atua apenas como descritor orquestrador e centralizador de configurações.

### 2.2. O Child POM (Submódulo)
Cada submódulo herda a referência do pai através da tag `<parent>`:

```xml
<parent>
    <groupId>guru.springframework</groupId>
    <artifactId>mb2g-mm-maven</artifactId>
    <version>${revision}</version>
</parent>

<artifactId>jpa-entities</artifactId>
<packaging>jar</packaging>
```
* **Herança Automática:** O filho herda todas as `<properties>`, plugins declarados em `<build>` e definições de `<dependencyManagement>` do Parent.

### 2.3. Dependências entre Módulos
Quando um módulo necessita consumir outro módulo do mesmo projeto (ex: `web-app` consumindo `jpa-entities` e `web-api`):

```xml
<dependencies>
    <dependency>
        <groupId>guru.springframework</groupId>
        <artifactId>jpa-entities</artifactId>
        <version>${project.version}</version>
    </dependency>
    <dependency>
        <groupId>guru.springframework</groupId>
        <artifactId>web-api</artifactId>
        <version>${project.version}</version>
    </dependency>
</dependencies>
```

---

## 3. Gestão Centralizada de Versões com `${revision}`

Em projetos com dezenas de submódulos, alterar a versão de cada `pom.xml` manualmente a cada release viola o princípio DRY (*Don't Repeat Yourself*).

A partir do Maven 3.5+, utiliza-se a propriedade especial **`${revision}`**:
1. No Parent POM:
   ```xml
   <version>${revision}</version>
   <properties>
       <revision>1.0-SNAPSHOT</revision>
   </properties>
   ```
2. Nos submódulos:
   ```xml
   <parent>
       <groupId>guru.springframework</groupId>
       <artifactId>mb2g-mm-maven</artifactId>
       <version>${revision}</version>
   </parent>
   ```
3. Nas dependências entre módulos irmãos:
   ```xml
   <version>${project.version}</version>
   ```

* **Vantagem em CI/CD:** É possível alterar a versão de toda a suíte multi-módulo em tempo de build na linha de comando:
  ```bash
  mvn clean install -Drevision=2.1.0-RC1
  ```

---

## 4. Higienização de Metadados com o Maven Flatten Plugin

### 4.1. O Problema das Variáveis nos Repositórios
Se publicarmos artefatos no repositório Maven (local ou remoto) que utilizem `${revision}` ou `${project.version}`, os descritores `.pom` instalados mantêm os placeholders não expandidos como texto literal (`<version>${revision}</version>`). Projetos externos que importarem essa biblioteca falharão porque o Maven consumidor não sabe o que é `${revision}`.

### 4.2. A Solução: `flatten-maven-plugin`
O plugin gera um arquivo `flattened-pom.xml` temporário onde todas as variáveis são resolvidas e substituídas pelo valor real antes da instalação/publicação:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.codehaus.mojo</groupId>
            <artifactId>flatten-maven-plugin</artifactId>
            <version>1.2.7</version>
            <configuration>
                <flattenMode>bom</flattenMode>
            </configuration>
            <executions>
                <execution>
                    <id>flatten</id>
                    <phase>process-resources</phase>
                    <goals> <goal>flatten</goal> </goals>
                </execution>
                <execution>
                    <id>flatten.clean</id>
                    <phase>clean</phase>
                    <goals> <goal>clean</goal> </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
* **Regra no `.gitignore`:** Adicione `.flattened-pom.xml` ao `.gitignore`, pois trata-se de um artefato gerado dinamicamente durante o build.

---

## 5. Protegendo o Ambiente com o Maven Enforcer Plugin

O **Maven Enforcer Plugin** bloqueia a execução do build caso requisitos fundamentais do ambiente (versão do Java ou do Maven) não sejam atendidos.

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-enforcer-plugin</artifactId>
            <version>3.4.1</version>
            <executions>
                <execution>
                    <id>enforce-versions</id>
                    <goals> <goal>enforce</goal> </goals>
                    <configuration>
                        <rules>
                            <!-- Requer Java 11 ou superior -->
                            <requireJavaVersion>
                                <version>[11,)</version>
                            </requireJavaVersion>
                            <!-- Requer Maven 3.5.0 ou superior (necessário para ${revision}) -->
                            <requireMavenVersion>
                                <version>[3.5.0,)</version>
                            </requireMavenVersion>
                        </rules>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 5.1. Sintaxe de Faixas de Versão (*Version Ranges*)
* `[11,)`: Maior ou igual a 11 (intervalo fechado à esquerda, aberto à direita).
* `(1.8, 11]`: Estritamente maior que 1.8 e menor ou igual a 11.
* `[3.6.0]`: Exatamente a versão 3.6.0.

---

## 6. Padronização de Dependências com BOM (*Bill of Materials*)

O termo **Bill of Materials (BOM)** vem da indústria de manufatura e representa a "lista completa de peças homologadas" para produzir um produto. No Maven, um BOM é um catálogo central de versões de dependências gerenciado via `<dependencyManagement>`.

### 6.1. Como Funciona o `<dependencyManagement>`?
* **Não inclui dependências:** Declarar uma biblioteca em `<dependencyManagement>` **NÃO adiciona o JAR** ao classpath nem o torna dependência transitiva do projeto.
* **Define a versão canônica:** Ele apenas estabelece: *"Se qualquer módulo filho ou dependência transitiva requisitar este `groupId:artifactId`, use EXATAMENTE esta versão declarada aqui"*.

### 6.2. Estrutura do BOM no Parent POM
```xml
<properties>
    <hibernate.version>5.4.2.Final</hibernate.version>
    <lombok.version>1.18.22</lombok.version>
    <mapstruct.version>1.4.2.Final</mapstruct.version>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>${hibernate.version}</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${mapstruct.version}</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 6.3. Consumo Descomplicado nos Submódulos
Nos submódulos (`jpa-entities`, `converters`), os desenvolvedores declaram apenas o `groupId` e o `artifactId`, omitindo qualquer `<version>`:

```xml
<dependencies>
    <!-- A versão é herdada automaticamente do BOM do Parent POM -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-core</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

---

## 7. Apêndice — Atualizações & Boas Práticas Modernas

### 1. CI-Friendly Versions Modernas: `${revision}${sha1}${changelist}`
A prática moderna de CI/CD para projetos multi-módulo adota as variáveis oficiais do Maven para versionamento dinâmico:
```xml
<version>${revision}${sha1}${changelist}</version>
<properties>
    <revision>1.2.0</revision>
    <sha1/>
    <changelist>-SNAPSHOT</changelist>
</properties>
```
* Em builds locais: a versão resulta em `1.2.0-SNAPSHOT`.
* Em pipelines de CI (GitHub Actions): executa-se `mvn deploy -Dchangelist= -Dsha1=.${GITHUB_SHA:0:7}`, gerando releases imutáveis rastreadas pelo hash do commit Git (ex: `1.2.0.a1b2c3d`).
* **Configuração Moderna do Flatten Plugin:** Para CI-Friendly Versions, recomenda-se:
  `<flattenMode>resolveCiFriendliesOnly</flattenMode>`.

### 2. Builds Paralelos no Reactor (`-T`)
Em máquinas com múltiplos núcleos e pipelines de CI com alta demanda, o Reactor suporta compilar módulos independentes em threads paralelas:
```bash
# Usa 4 threads dedicadas:
mvn clean install -T 4

# Aloca 1 thread por núcleo de CPU da máquina:
mvn clean install -T 1C
```

### 3. Comandos Avançados de Filtragem do Reactor (`-pl`, `-am`, `-amd`)
Em projetos multi-módulos corporativos com dezenas de módulos, reconstruir todo o projeto para testar uma alteração em um único submódulo é inviável. Comandos úteis:
```bash
# Compila APENAS o submódulo web-app e tudo que ele precisa (Also Make -am):
mvn clean install -pl web-app -am

# Executa testes no web-api e em todos os módulos que dependem dele (Also Make Dependents -amd):
mvn test -pl web-api -amd
```

### 4. Maven 4: Separação Nativa entre Build POM e Consumer POM
No Maven 4, a dependência do `flatten-maven-plugin` deixa de existir. O próprio núcleo do Maven passa a separar nativamente o **Build POM** (usado pelo desenvolvedor localmente com interpolação de `${revision}`) do **Consumer POM** (versão sanitizada e estática publicada nos repositórios para consumo público).
