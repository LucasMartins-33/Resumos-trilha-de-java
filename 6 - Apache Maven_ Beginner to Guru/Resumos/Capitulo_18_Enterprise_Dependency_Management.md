# Capítulo 18: Enterprise Dependency Management

Neste capítulo, estudamos o padrão arquitetural de **Gestão Corporativa de Dependências (*Enterprise Dependency Management*)**, explorando a criação de **BOMs (*Bill of Materials*)** e Parent POMs corporativos em camadas para ecossistemas de microsserviços, a padronização de plugins de build, a imposição de regras de governança com o **Maven Enforcer Plugin** e estratégias de produtividade com múltiplos projetos no IntelliJ IDEA.

---

## 1. Conceitos Fundamentais

### 1.1 O Desafio das Dependências em Arquiteturas de Microsserviços
Em organizações de médio e grande porte com dezenas de microsserviços e múltiplas equipes de engenharia:
1. **Proliferação e Divergência de Versões**: Cada equipe acaba definindo suas próprias versões de bibliotecas fundamentais (Spring Boot, Jackson, Hibernate, MapStruct, JUnit), criando um ecossistema caótico e difícil de manter.
2. **Riscos de Segurança e Vulnerabilidades (CVEs)**: Quando uma vulnerabilidade crítica é descoberta em uma biblioteca comum (como uma brecha de desserialização no Jackson ou no Log4j), atualizar o `pom.xml` manualmente em dezenas de repositórios independentes é lento e propenso a erros.
3. **Auditoria e Conformidade (PCI-DSS, SOX, SAS 70 / SOC 2)**: Órgãos reguladores exigem processos formais comprovando quais bibliotecas estão em produção e como a esteira garante que artefatos vulneráveis não sejam publicados.

---

### 1.2 O Conceito de BOM (*Bill of Materials*)
O termo **BOM** é originário da engenharia industrial e manufatura (a "lista de peças" ou receita necessária para fabricar um equipamento complexo, como um motor):
* No Maven, um **BOM** é um projeto de empacotamento especial (`<packaging>pom</packaging>`) que define um catálogo centralizado, curado e testado de bibliotecas e versões compatíveis.
* **Eliminação de Duplicação**: Os microsserviços herdam ou importam o BOM, permitindo declarar dependências **sem precisar informar a tag `<version>`**.

---

### 1.3 Arquitetura de Herança em Camadas (*Layered BOMs*)
Em vez de um único arquivo monolítico, adota-se uma cadeia modular de herança:

```text
               ╔═══════════════════════════════════════╗
               ║      spring-boot-starter-parent       ║
               ║ (BOM oficial do Spring Boot/Pivotal)  ║
               ╚═══════════════════════════════════════╝
                                   ▲
                                   │ Herda
               ╔═══════════════════════════════════════╗
               ║          sfg-beer-works-bom           ║
               ║ (BOM Corporativo Raiz da Organização) ║
               ║ • Java 17/21 Baseline                 ║
               ║ • UTF-8 e Enforcer Plugin Rules       ║
               ║ • Lombok, JUnit 5 e MapStruct Plugin  ║
               ╚═══════════════════════════════════════╝
                                   ▲
                                   │ Herda
               ╔═══════════════════════════════════════╗
               ║           sfg-brewery-bom             ║
               ║ (Especialização do Domínio de Negócio)║
               ║ • Spring Data JPA & MySQL Driver      ║
               ║ • Spring Boot Starter Web & Cache     ║
               ╚═══════════════════════════════════════╝
                    ▲              ▲              ▲
            Herda   │      Herda   │      Herda   │
        ┌───────────┴──┐  ┌────────┴──────┐  ┌────┴────────────┐
        │ beer-service │  │ order-service │  │inventory-service│
        └──────────────┘  └───────────────┘  └─────────────────┘
```

#### Benefício da Estrutura:
Os arquivos `pom.xml` dos microsserviços individuais tornam-se incrivelmente compactos (reduzidos de centenas de linhas para meras dezenas), contendo apenas as declarações estritamente exclusivas do serviço.

---

### 1.4 `<dependencyManagement>` vs. `<dependencies>` no BOM

| Elemento no BOM | Comportamento no Microsserviço Filho |
| :--- | :--- |
| **`<dependencyManagement>`** | **Opcional**: Apenas prescreve a versão e escopo. O microsserviço filho **não** recebe o JAR no classpath a menos que declare a dependência no seu próprio POM (sem `<version>`). |
| **`<dependencies>`** | **Obrigatório/Universal**: Força a inclusão direta da dependência em **todos** os projetos que herdarem o BOM (ideal para ferramentas universais como Actuator, Lombok, MapStruct e Testes). |

---

### 1.5 Governança Estrita com o `maven-enforcer-plugin`
O plugin de imposição do Maven permite abortar o build imediatamente caso o ambiente de desenvolvimento ou o código viole as regras corporativas:
* **`requireMavenVersion`**: Garante que o desenvolvedor e o CI estejam em uma versão homologada do Maven (ex: `[3.6.0,)`).
* **`requireJavaVersion`**: Garante conformidade com o JDK LTS da empresa (ex: Java 11, 17 ou 21).
* **`requireReleaseDeps`**: Impede que builds de release final dependam acidentalmente de bibliotecas SNAPSHOT.

---

## 2. Sintaxe, Comandos & Configurações

### 2.1 Estrutura do Parent BOM Corporativo (`pom.xml`)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.4</version>
        <relativePath/>
    </parent>

    <groupId>com.empresa.bom</groupId>
    <artifactId>enterprise-parent-bom</artifactId>
    <version>1.0.0</version>
    <!-- Mandatório: BOMs não geram JARs de código, apenas metadados POM -->
    <packaging>pom</packaging>

    <properties>
        <java.version>17</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <mapstruct.version>1.5.5.Final</mapstruct.version>
        <lombok-mapstruct-binding.version>0.2.0</lombok-mapstruct-binding.version>
    </properties>

    <!-- 1. Catálogo de Versões Curadas (Herança Opcional de Versão) -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.mapstruct</groupId>
                <artifactId>mapstruct</artifactId>
                <version>${mapstruct.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <!-- 2. Dependências Universais (Injetadas em TODOS os microsserviços) -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Limpeza automática no início de cada build -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-clean-plugin</artifactId>
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

            <!-- Configuração Centralizada de Compilação (Lombok + MapStruct) -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${mapstruct.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok-mapstruct-binding</artifactId>
                            <version>${lombok-mapstruct-binding.version}</version>
                        </path>
                    </annotationProcessorPaths>
                    <compilerArgs>
                        <compilerArg>-Amapstruct.defaultComponentModel=spring</compilerArg>
                    </compilerArgs>
                </configuration>
            </plugin>

            <!-- Regras de Imposição e Governança -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-enforcer-plugin</artifactId>
                <version>3.4.1</version>
                <executions>
                    <execution>
                        <id>enforce-rules</id>
                        <goals>
                            <goal>enforce</goal>
                        </goals>
                        <configuration>
                            <rules>
                                <!-- Maven >= 3.6.3 -->
                                <requireMavenVersion>
                                    <version>[3.6.3,)</version>
                                </requireMavenVersion>
                                <!-- JDK 17 ou superior -->
                                <requireJavaVersion>
                                    <version>[17,)</version>
                                </requireJavaVersion>
                                <!-- Proíbe dependências SNAPSHOT em releases -->
                                <requireReleaseDeps>
                                    <onlyWhenRelease>true</onlyWhenRelease>
                                </requireReleaseDeps>
                            </rules>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

---

### 2.2 O POM Ultra-Enxuto do Microsserviço Consumidor
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.empresa.bom</groupId>
        <artifactId>enterprise-parent-bom</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>beer-service</artifactId>
    <!-- Versão e groupId herdados do parent -->

    <!-- Declara apenas dependências específicas deste serviço (sem versão!) -->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
        </dependency>
    </dependencies>
    <!-- Zero blocos <build> repetitivos! Tudo é herdado do BOM -->
</project>
```

---

## 3. Dicas de Produtividade no IntelliJ IDEA

* **Múltiplos Projetos Independentes no Mesmo Workspace**:
  1. No Git, mantenha os microsserviços em repositórios independentes (`beer-service`, `order-service`, `inventory-service`).
  2. No IntelliJ, crie um **Empty Project** (`File -> New -> Project -> Empty Project`).
  3. Importe cada serviço usando: `File -> New -> Module from Existing Sources...` selecionando o `pom.xml` de cada um.
  4. **Vantagem**: Permite navegar, refatorar e executar testes em todos os microsserviços simultaneamente a partir de uma única janela, sem precisar alternar instâncias do IntelliJ e sem transformá-los em um monorepo no Git.

---

## 4. Apêndice — Atualizações & Boas Práticas Modernas

### 4.1 Herança (`<parent>`) vs. Importação de BOM (`<scope>import</scope>`)
O padrão de herança via `<parent>` utilizado no curso possui uma limitação inerente: **o Maven não suporta herança múltipla** de pais. Se o microsserviço herda do BOM da empresa, ele não pode herdar de nenhum outro parent.
* **Padrão Moderno Recomendado (Importação de BOM)**:
  Em vez de herança direta, utiliza-se a importação no `<dependencyManagement>`:
  ```xml
  <dependencyManagement>
      <dependencies>
          <!-- Importa o BOM do Spring Boot -->
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-dependencies</artifactId>
              <version>3.2.4</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
          <!-- Importa o BOM Corporativo -->
          <dependency>
              <groupId>com.empresa.bom</groupId>
              <artifactId>enterprise-bom</artifactId>
              <version>2.0.0</version>
              <type>pom</type>
              <scope>import</scope>
          </dependency>
      </dependencies>
  </dependencyManagement>
  ```
  Isso permite compor múltiplos catálogos de dependências de forma limpa e flexível.

### 4.2 Gestão Automatizada de Vulnerabilidades (Dependabot / Renovate)
No desenvolvimento moderno, o BOM corporativo é monitorado por robôs:
* **Renovate / Dependabot**: Monitora lançamentos no Maven Central e abre Pull Requests automáticos no repositório do BOM corporativo sempre que há patches de segurança em dependências gerenciadas.
* Uma vez que a equipe aprova e publica a nova versão do BOM (`2.0.1`), todos os microsserviços recebem a atualização centralizada.

### 4.3 Do BOM ao SBOM (Software Bill of Materials)
Devido a novas regulamentações internacionais de cibersegurança (como as ordens executivas do governo dos EUA e normas europeias):
* Toda organização moderna é incentivada a gerar um **SBOM** em formato padronizado (**CycloneDX** ou **SPDX**) durante a compilação via `cyclonedx-maven-plugin`.
* O SBOM é um inventário legível por máquina de todas as dependências, licenças e hashes criptográficos contidos no produto final entregue em produção.
