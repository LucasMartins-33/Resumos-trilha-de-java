# Capítulo 01: Spring Boot 4 Quick Start

Este resumo abrange os conceitos fundamentais do Spring Boot 4 / Spring 7 e Hibernate, cobrindo a visão geral do ecossistema, configuração do ambiente, criação de aplicações via Spring Initializr, estrutura do Maven, criação de Controllers REST, ferramentas de desenvolvimento (DevTools), monitoramento (Actuator), segurança básica, execução via linha de comando e gerenciamento de propriedades de aplicação.

---

## 📌 1. Visão Geral do Spring Boot e Ecossistema

### O Problema do Spring Tradicional
Antes do Spring Boot, configurar uma aplicação Spring exigia:
* Selecionar e declarar manualmente dezenas de dependências JAR compatíveis no `pom.xml` ou `build.gradle`.
* Escrever extensas configurações em XML ou Java Config (`@Configuration`, `@Bean`).
* Instalar, configurar e gerenciar servidores web externos (como Tomcat, Jetty ou JBoss/WildFly) separadamente.

### A Solução: Spring Boot
O Spring Boot simplifica a criação de aplicações Spring autônomas e prontas para produção:
* **Autoconfiguração (`Auto-Configuration`)**: O Spring Boot configura automaticamente o ecossistema com base nas dependências encontradas no *classpath* e nos arquivos de propriedades.
* **Servidor HTTP Embutido**: Inclui por padrão o Tomcat embutido (com suporte fácil a Jetty ou Undertow), eliminando a necessidade de implantar arquivos WAR em servidores externos.
* **Resolução de Conflitos de Dependências**: Através dos **Spring Boot Starters**, garante que todas as bibliotecas utilizadas sejam 100% compatíveis entre si.
* **Não é substituto do Spring**: O Spring Boot utiliza o Spring Framework por trás dos panos (`Spring Core`, `Spring MVC`, `Spring Security`, etc.). Ele apenas elimina a complexidade da configuração.

---

## 📌 2. Pré-requisitos e Ambiente de Desenvolvimento

* **Java Development Kit (JDK)**: Requer **JDK 17 ou superior** (versões modernas utilizam Java 17, 21 ou superior).
* **IDE Java**: IntelliJ IDEA (Community ou Ultimate), Eclipse STS (Spring Tool Suite) ou VS Code.
* **Build Tool**: Apache Maven (ou Gradle).

---

## 📌 3. Spring Initializr e Estrutura do Projeto Maven

O [start.spring.io](https://start.spring.io) é a ferramenta oficial para inicializar projetos Spring Boot rapidamente.

### Estrutura Padrão do Diretório Maven (`Maven Standard Directory Structure`)

```text
meu-projeto/
├── pom.xml                               # Arquivo principal de configuração do Maven (Shopping List)
├── mvnw / mvnw.cmd                       # Maven Wrapper (scripts para rodar o Maven sem instalá-lo)
└── src/
    ├── main/
    │   ├── java/                         # Código fonte Java (pacotes e classes)
    │   └── resources/                    # Arquivos de configuração e recursos estáticos
    │       ├── application.properties   # Propriedades de configuração do Spring Boot
    │       ├── static/                   # Recursos estáticos (HTML, CSS, JS, Imagens)
    │       └── templates/                # Templates para renderização no servidor (Thymeleaf, FreeMarker)
    └── test/
        └── java/                         # Código fonte dos testes unitários e de integração
```

> [!WARNING]
> **Atenção quanto à pasta `src/main/webapp`**: No Spring Boot empacotado como **JAR** (com Tomcat embutido), a pasta `src/main/webapp` é ignorada pelos mecanismos de build. Para conteúdos estáticos em empacotamento JAR, utilize **`src/main/resources/static`**.

---

## 📌 4. Sintaxe e Anotações Principais do Capítulo

### 1. `@SpringBootApplication`
Anotação composta que fica na classe principal (ponto de entrada) da aplicação.
* **Localização**: Pacote raiz da aplicação.
* **Composição**: Inclui `@Configuration`, `@EnableAutoConfiguration` e `@ComponentScan`.

#### Sintaxe / Exemplo de Código:
```java
package com.luv2code.springboot.demo.mycoolapp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MycoolappApplication {

    public static void main(String[] args) {
        // Inicializa o contexto da aplicação Spring e o servidor Tomcat embutido
        SpringApplication.run(MycoolappApplication.class, args);
    }
}
```

---

### 2. `@RestController` & `@GetMapping`
Utilizados para expor endpoints REST HTTP na aplicação.

#### Explicação de Sintaxe:
* `@RestController`: Combinação de `@Controller` e `@ResponseBody`. Indica que a classe tratará requisições HTTP e retornará a resposta diretamente no corpo da mensagem (geralmente formatada em JSON ou texto puro).
* `@GetMapping("/caminho")`: Mapeia requisições HTTP do tipo `GET` para o método anotado.

#### Sintaxe / Exemplo de Código:
```java
package com.luv2code.springboot.demo.mycoolapp.rest;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FunRestController {

    // Expõe o endpoint GET "/" que retorna "Hello World!"
    @GetMapping("/")
    public String sayHello() {
        return "Hello World!";
    }

    // Expõe o endpoint GET "/workout"
    @GetMapping("/workout")
    public String getDailyWorkout() {
        return "Run a hard 5k!";
    }

    // Expõe o endpoint GET "/fortune"
    @GetMapping("/fortune")
    public String getDailyFortune() {
        return "Today is your lucky day!";
    }
}
```

---

### 3. `@Value`
Utilizada para injetar valores de propriedades definidas no arquivo `application.properties` (ou variáveis de ambiente) diretamente em campos das classes gerenciadas pelo Spring.

#### Sintaxe:
```java
@Value("${nome.da.propriedade:valor_padrao_opcional}")
private String meuCampo;
```

#### Exemplo de Código:

**No arquivo `src/main/resources/application.properties`:**
```properties
coach.name=Mickey Mouse
team.name=The Mouse Club
```

**Na classe Java (`FunRestController.java`):**
```java
package com.luv2code.springboot.demo.mycoolapp.rest;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class FunRestController {

    // Injeta a propriedade coach.name
    @Value("${coach.name}")
    private String coachName;

    // Injeta a propriedade team.name
    @Value("${team.name}")
    private String teamName;

    @GetMapping("/teaminfo")
    public String getTeamInfo() {
        return "Coach: " + coachName + ", Team name: " + teamName;
    }
}
```

---

## 📌 5. Conceitos-Chave do Apache Maven

### O que é o POM (`pom.xml`)?
O `pom.xml` (**Project Object Model**) é o arquivo central de configuração do Maven. Funciona como a "lista de compras" do projeto.

### Coordenadas GAV (`Group ID`, `Artifact ID`, `Version`)
Identificam de forma única um projeto ou biblioteca no ecossistema Java:
* **`groupId`**: Nome da organização/empresa em formato de pacote reverso (ex: `com.luv2code.springboot`).
* **`artifactId`**: Nome do módulo/projeto (ex: `mycoolapp`).
* **`version`**: Versão da aplicação (ex: `1.0.0-SNAPSHOT` para desenvolvimento, ou `1.0.0` para final).

```xml
<groupId>com.luv2code.springboot.demo</groupId>
<artifactId>mycoolapp</artifactId>
<version>0.0.1-SNAPSHOT</version>
<name>mycoolapp</name>
<description>Demo project for Spring Boot</description>
```

### Spring Boot Starter Parent
Define as configurações padrão do Maven para o Spring Boot (versão do Java, codificação UTF-8, gerenciamento centralizado de versões de dependências e configuração do plugin de build).

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.5</version> <!-- Utilize a versão estável atual -->
    <relativePath/> <!-- busca a partir do repositório central -->
</parent>
```

### Spring Boot Starters
São coleções organizadas de dependências curadas pela equipe do Spring. Ao adicionar um *Starter*, você não precisa declarar manualmente cada biblioteca e suas respectivas versões.

#### Principais Starters do Capítulo:
1. **`spring-boot-starter-web`**: Inclui Spring MVC, Tomcat embutido, Jackson (JSON) e suporte a REST.
2. **`spring-boot-starter-actuator`**: Inclui recursos de monitoramento, métricas e checagem de saúde da aplicação.
3. **`spring-boot-starter-security`**: Inclui o Spring Security para autenticação e autorização.
4. **`spring-boot-devtools`**: Reinicia automaticamente a aplicação sempre que arquivos no classpath forem modificados.

#### Exemplo de Declaração de Starters no `pom.xml`:
```xml
<dependencies>
    <!-- Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- DevTools -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>

    <!-- Actuator -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!-- Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
</dependencies>
```

---

## 📌 6. Spring Boot DevTools (Recarregamento Automático)

O `spring-boot-devtools` monitora o classpath e reinicia a aplicação automaticamente quando os arquivos de código compilados (`.class`) mudam.

### Configuração Extra Obrigatória no IntelliJ IDEA (Community Edition)
Para funcionar no IntelliJ Community, é necessário realizar duas configurações na IDE:
1. Ir em **Settings / Preferences** $\rightarrow$ **Build, Execution, Deployment** $\rightarrow$ **Compiler**:
   * Marcar a opção **`Build project automatically`**.
2. Ir em **Settings / Preferences** $\rightarrow$ **Advanced Settings**:
   * Marcar a opção **`Allow auto-make to start even if developed application is currently running`**.

---

## 📌 7. Spring Boot Actuator (Monitoramento & Métricas)

O Actuator expõe endpoints HTTP para monitorar a saúde, configurações e métricas da aplicação em ambiente de produção/DevOps.

### Endpoints Principais
* `/actuator/health`: Retorna o status de saúde da aplicação (`UP`, `DOWN`).
* `/actuator/info`: Exibe informações customizadas da aplicação.
* `/actuator/beans`: Lista todos os Beans registrados no Spring Application Context.
* `/actuator/mappings`: Lista todos os caminhos/URL mappings de requisições.
* `/actuator/threaddump`: Exibe o dump de threads em execução.

### Configurando e Expondo Endpoints em `application.properties`

> [!NOTE]
> **Atualização de Versão (Spring Boot 3 / 4)**: 
> Nas versões modernas do Spring Boot, por padrão apenas o endpoint `/health` fica exposto via Web. O uso da propriedade `management.info.env.enabled=true` é necessário para permitir que variáveis de ambiente/propriedades preencham o `/info`.

```properties
# Expor endpoints específicos (ex: health e info)
management.endpoints.web.exposure.include=health,info

# Expor TODOS os endpoints via Web (Usar com cuidado!)
management.endpoints.web.exposure.include=*

# Excluir endpoints específicos
management.endpoints.web.exposure.exclude=health,info

# Habilitar exibição de propriedades env no endpoint /info
management.info.env.enabled=true

# Customizando informações do /info
info.app.name=My Super Cool App
info.app.description=Aplicação de Aprendizado Spring Boot
info.app.version=1.0.0
```

### Protegendo os Endpoints do Actuator com Spring Security
Ao adicionar a dependência `spring-boot-starter-security`:
* Todos os endpoints do Actuator (exceto `/health` se configurado publicamente) passam a exigir autenticação **HTTP Basic**.
* O usuário padrão é `user` e a senha padrão é impressa no log de inicialização no console (`Using generated security password: ...`).

#### Definindo Usuário e Senha Personalizados em `application.properties`:
```properties
spring.security.user.name=admin
spring.security.user.password=topsecret
```

---

## 📌 8. Execução de Aplicações Spring Boot pela Linha de Comando

Aplicações Spring Boot empacotadas como JAR possuem o servidor Tomcat embutido e podem ser executadas independentemente de qualquer IDE ou servidor externo.

### Abordagem 1: Executando o arquivo JAR empacotado

1. **Gerar o arquivo JAR no diretório `target/`**:
   * Windows: `mvnw package`
   * Mac/Linux: `./mvnw package`
2. **Executar o arquivo JAR**:
   ```bash
   java -jar target/mycoolapp-0.0.1-SNAPSHOT.jar
   ```

### Abordagem 2: Utilizando o Plugin do Spring Boot Maven

Executa a aplicação diretamente a partir do código fonte:
* Windows: `mvnw spring-boot:run`
* Mac/Linux: `./mvnw spring-boot:run`
* (Se o Maven estiver instalado globalmente no SO, pode-se usar diretamente `mvn spring-boot:run`).

---

## 📌 9. Configurações Comuns do Arquivo `application.properties`

O Spring Boot possui centenas de propriedades pré-definidas. Abaixo estão as mais utilizadas no capítulo 1:

```properties
# ==========================================
# 1. CONFIGURAÇÕES DO SERVIDOR HTTP (WEB)
# ==========================================
# Alterar a porta do servidor embutido (Padrão: 8080)
server.port=7070

# Definir Context Path (Prefixo global da aplicação)
server.servlet.context-path=/mycoolapp

# Session Timeout (Exemplo: 15 minutos)
server.servlet.session.timeout=15m

# ==========================================
# 2. NÍVEL DE LOGS (LOGGING)
# ==========================================
# Definir níveis de log por pacote (TRACE, DEBUG, INFO, WARN, ERROR, FATAL, OFF)
logging.level.org.springframework=DEBUG
logging.level.org.hibernate=TRACE
logging.level.com.luv2code=INFO

# Enviar logs para arquivo físico ao invés de apenas o console
logging.file.name=my-crazy-stuff.log

# ==========================================
# 3. PROPRIEDADES CUSTOMIZADAS (@Value)
# ==========================================
coach.name=Mickey Mouse
team.name=The Mouse Club
```

> [!TIP]
> **Atualização de Sintaxe (Spring Boot 3/4)**:
> Note que em versões recentes a propriedade do context-path mudou de `server.context-path` (versões legadas) para **`server.servlet.context-path`**.

---

## 📋 Resumo Rápido para Consulta Futura

| Conceito | Anotação / Comando / Propriedade | Descrição |
| :--- | :--- | :--- |
| **Classe Principal** | `@SpringBootApplication` | Inicializa a aplicação Spring e o Tomcat embutido. |
| **Controller REST** | `@RestController` | Define uma classe como controladora de APIs REST. |
| **Mapeamento GET** | `@GetMapping("/endpoint")` | Mapeia chamadas HTTP GET para um método. |
| **Injeção de Propriedades** | `@Value("${chave}")` | Injeta valores do `application.properties` na classe. |
| **Build/Empacotamento** | `mvnw package` | Compila e gera o arquivo `.jar` na pasta `target/`. |
| **Executar JAR** | `java -jar app.jar` | Roda a aplicação de forma autônoma via linha de comando. |
| **Executar via Maven** | `mvnw spring-boot:run` | Roda a aplicação diretamente dos fontes. |
| **Porta do Servidor** | `server.port=8080` | Define a porta do Tomcat embutido. |
| **Context Path** | `server.servlet.context-path=/app` | Define o prefixo de URL para a aplicação. |
| **Endpoints Actuator** | `management.endpoints.web.exposure.include=*` | Define quais métricas do Actuator expor via HTTP. |
