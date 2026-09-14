# Questões Práticas – Capítulo 01: Spring Boot 4 Quick Start

## Exercício 1.1 – Criar um projeto Spring Boot usando o Spring Initializr
**Cenário**
Você precisa iniciar um novo projeto Spring Boot 4 stand‑alone (arquivo JAR) via Spring Initializr.

1️⃣ Crie o `pom.xml` com a dependência `spring-boot-starter-parent` na versão **4.0.0**.
2️⃣ Inclua o `spring-boot-starter-web` e o `spring-boot-starter-actuator`.
3️⃣ Defina a propriedade `java.version` como **17**.

```xml
<project ...>
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>4.0.0</version>
        <relativePath/>
    </parent>
    <properties>
        <java.version>17</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## Exercício 1.2 – Classe principal com `@SpringBootApplication`
**Cenário**
Crie a classe de bootstrap que inicia a aplicação.

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // 🟢 Atividade: anotação principal
public class DemoApplication {
    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

---

## Exercício 1.3 – Configurar porta customizada via `application.properties`
**Cenário**
Altere a porta padrão para **8085**.

```properties
# application.properties
server.port=8085 // 🟢 Atividade: definir porta
```

---

## Exercício 1.4 – Expor endpoint `/api/hello` que devolve um JSON
**Cenário**
Crie um `@RestController` que responda a **GET** `/api/hello` retornando `{"message":"Hello Spring Boot 4!"}`.

```java
package com.example.demo;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.Map;

@RestController
@RequestMapping("/api") // 🟢 Atividade: base path
public class HelloController {

    @GetMapping("/hello") // 🟢 Atividade: método GET
    public Map<String, String> hello() {
        return Map.of("message", "Hello Spring Boot 4!");
    }
}
```

---

## Exercício 1.5 – Ativar Spring DevTools para reinicialização automática
**Cenário**
Adicione a dependência `spring-boot-devtools` em modo *runtime* e verifique que alterações nas classes reiniciam o contexto.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
</dependency>
```

---

## Exercício 1.6 – Expor o endpoint de **Actuator** `/actuator/health`
**Cenário**
Faça um `curl` para `http://localhost:8085/actuator/health` e confirme que o retorno contém `"status":"UP"`.

---

## Exercício 1.7 – Criar um *profile* `dev` que habilita o **H2 Console**
**Cenário**
No `application-dev.properties` adicione as propriedades necessárias para habilitar o console web do H2 (`/h2-console`).

```properties
spring.h2.console.enabled=true
spring.h2.console.path=/h2-console
```

---

## Exercício 1.8 – Testar a aplicação com JUnit 5
**Cenário**
Escreva um teste de integração que carregue o contexto Spring e valide que o bean `HelloController` está presente.

```java
@SpringBootTest
class DemoApplicationTests {
    @Autowired
    private HelloController controller;

    @Test
    void contextLoads() {
        Assertions.assertNotNull(controller);
    }
}
```

---

## Exercício 1.9 – Empacotar como **executable JAR**
**Cenário**
Execute `mvn clean package` e verifique que o artefato `demo-0.0.1-SNAPSHOT.jar` contém a classe `DemoApplication` e pode ser iniciado com `java -jar target/demo-0.0.1-SNAPSHOT.jar`.

---

## Exercício 1.10 – Deploy rápido usando **Dockerfile**
**Cenário**
Escreva um `Dockerfile` que use a imagem `eclipse-temurin:17-jre` e copie o JAR criado.

```dockerfile
FROM eclipse-temurin:17-jre
ARG JAR_FILE=target/*.jar
COPY ${JAR_FILE} app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

---

**Como usar**
1. Crie a estrutura de diretórios (`src/main/java/com/example/demo`).
2. Preencha cada *TODO* marcado com 🟢 ou 🟡.
3. Rode `mvn spring-boot:run` e teste os endpoints.

> **Arquivo salvo em** `C:\Users\lucas\OneDrive\Documentos\Cursos\spring-boot-4-spring-7-hibernate-for-beginners\Questoes\Pratica\Capitulo_01_QuickStart_Pratico_Questoes.md`
