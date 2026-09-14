# 📝 Questões Práticas - Capítulo 01: Spring Boot 4 Quick Start

Este documento contém 10 exercícios práticos baseados nos conceitos do **Capítulo 01 (Spring Boot 4 Quick Start / Visão Geral, Configurações, DevTools, Actuator e Linha de Comando)**.

---

## 🎯 Questões Práticas

### **Questão 1: Criando um Controller REST e Endpoints GET**
**Cenário:** Você precisa criar uma API REST simples para um sistema escolar.
**Tarefa:**
1. Crie uma classe chamada `StudentRestController` no pacote `com.escola.demo.rest`.
2. Anote a classe para que ela funcione como um Controller REST.
3. Crie dois endpoints HTTP `GET`:
   - `/` : Retorna a mensagem `"Bem-vindo ao Sistema Escolar!"`.
   - `/curso` : Retorna o texto `"Curso de Spring Boot 4 e Hibernate"`.

---

### **Questão 2: Injeção de Propriedades Customizadas com `@Value`**
**Cenário:** As informações de contato do suporte da sua aplicação devem ser configuráveis externamente no arquivo de propriedades sem alterar o código Java.
**Tarefa:**
1. No arquivo `src/main/resources/application.properties`, adicione duas chaves:
   - `suporte.email=suporte@escola.com`
   - `suporte.telefone=0800-123-4567`
2. No seu `StudentRestController`, utilize a anotação `@Value` para injetar essas duas propriedades em atributos privados (`emailSuporte` e `telefoneSuporte`).
3. Crie um endpoint GET `/contato` que retorne uma String formatada: `"Suporte: suporte@escola.com | Tel: 0800-123-4567"`.

---

### **Questão 3: Alterando Porta e Context Path do Servidor Embutido**
**Cenário:** A porta padrão `8080` já está sendo utilizada por outra aplicação na máquina de desenvolvimento e a equipe solicitou que todas as URLs da aplicação tenham o prefixo `/api`.
**Tarefa:**
Escreva as linhas de configuração corretas no arquivo `application.properties` para:
1. Alterar a porta do servidor Tomcat embutido para `9090`.
2. Definir o *Context Path* global da aplicação para `/api`.
3. Indique como ficará a URL completa para acessar o endpoint `/contato` criado na Questão 2.

---

### **Questão 4: Configuração e Dependência do Spring Boot Actuator**
**Cenário:** O time de DevOps precisa monitorar o status de saúde e informações da aplicação Spring Boot.
**Tarefa:**
1. Qual dependência (Starter) do Maven deve ser adicionada ao `pom.xml` para habilitar o Actuator? Escreva o bloco XML `<dependency>`.
2. No arquivo `application.properties`, configure a aplicação para expor via Web os endpoints `/actuator/health` e `/actuator/info`.
3. Habilite a propriedade necessária para permitir que informações do ambiente preencham o endpoint `/info`.

---

### **Questão 5: Customizando Informações da Aplicação no Endpoint `/info`**
**Cenário:** Você deseja expor o nome da aplicação, a versão e o nome do desenvolvedor através do endpoint `/actuator/info`.
**Tarefa:**
Escreva as propriedades necessárias no `application.properties` para definir:
- Nome da aplicação: `Sistema de Gestao Academica`
- Versão: `2.1.0`
- Desenvolvedor: `Lucas`

---

### **Questão 6: Protegendo Actuator e Endpoints com Spring Security**
**Cenário:** Para evitar acessos não autorizados aos endpoints de monitoramento em produção, você adicionou o `spring-boot-starter-security`.
**Tarefa:**
1. O que acontece com o acesso aos endpoints REST quando essa dependência é adicionada ao projeto?
2. Configurar no `application.properties` um nome de usuário (`admin`) e uma senha personalizada (`senhaSegura123`) para a autenticação HTTP Basic.

---

### **Questão 7: Habilitando Recarregamento Automático com DevTools no IntelliJ IDEA**
**Cenário:** Você adicionou a dependência `spring-boot-devtools` no `pom.xml`, mas percebeu que ao alterar o código no IntelliJ IDEA (Community Edition), a aplicação não reinicia automaticamente.
**Tarefa:**
Quais são as duas configurações obrigatórias que devem ser marcadas nas preferências/configurações do IntelliJ IDEA para que o DevTools funcione corretamente durante a execução da aplicação?

---

### **Questão 8: Gerenciamento de Logs no `application.properties`**
**Cenário:** Durante a depuração de um problema com o Spring Framework, você precisa aumentar o nível de detalhe dos logs e também salvar esses logs em um arquivo físico no disco.
**Tarefa:**
Escreva as configurações no `application.properties` para:
1. Definir o nível de log do pacote `org.springframework` para `DEBUG`.
2. Definir o nível de log do pacote da sua aplicação (`com.escola.demo`) para `TRACE`.
3. Redirecionar os logs para um arquivo chamado `escola-debug.log`.

---

### **Questão 9: Execução da Aplicação via Linha de Comando (CLI)**
**Cenário:** Você está em um servidor Linux sem IDE instalada e precisa compilar e executar o projeto Spring Boot diretamente pelo terminal.
**Tarefa:**
1. Escreva o comando do Maven Wrapper (`./mvnw`) para gerar o arquivo `.jar` executável na pasta `target/`.
2. Escreva o comando Java (`java -jar`) para iniciar a aplicação compilada (assuma que o nome do arquivo gerado seja `demo-0.0.1-SNAPSHOT.jar`).
3. Qual é o comando alternativo do Maven Wrapper para compilar e rodar a aplicação em um único passo sem gerar o pacote explicitamente?

---

### **Questão 10: Estrutura da Classe Principal `@SpringBootApplication`**
**Cenário:** Um desenvolvedor júnior criou a classe principal do Spring Boot e a colocou dentro do pacote `com.escola.demo.controller.main`, enquanto os Controllers estão no pacote `com.escola.demo.rest`. Ao rodar a aplicação, os endpoints retornavam **404 Not Found**.
**Tarefa:**
1. Explique por que os endpoints não foram encontrados pelo Spring Boot. (Dica: pense no funcionamento do `@ComponentScan` contido no `@SpringBootApplication`).
2. Onde a classe principal deve ser posicionada para resolver esse problema e qual é o código correto dessa classe principal em Java?

---

<br/>

---

## 🔑 Gabarito e Resolução dos Exercícios

### **Resolução 1**
```java
package com.escola.demo.rest;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class StudentRestController {

    @GetMapping("/")
    public String sayHello() {
        return "Bem-vindo ao Sistema Escolar!";
    }

    @GetMapping("/curso")
    public String getCursoInfo() {
        return "Curso de Spring Boot 4 e Hibernate";
    }
}
```

---

### **Resolução 2**

**1. `src/main/resources/application.properties`:**
```properties
suporte.email=suporte@escola.com
suporte.telefone=0800-123-4567
```

**2 & 3. `StudentRestController.java`:**
```java
package com.escola.demo.rest;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class StudentRestController {

    @Value("${suporte.email}")
    private String emailSuporte;

    @Value("${suporte.telefone}")
    private String telefoneSuporte;

    @GetMapping("/contato")
    public String getContato() {
        return "Suporte: " + emailSuporte + " | Tel: " + telefoneSuporte;
    }
}
```

---

### **Resolução 3**

**1 & 2. Configuração em `application.properties`:**
```properties
server.port=9090
server.servlet.context-path=/api
```

**3. URL Completa:**
`http://localhost:9090/api/contato`

---

### **Resolução 4**

**1. Dependência no `pom.xml`:**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**2 & 3. Configurações em `application.properties`:**
```properties
management.endpoints.web.exposure.include=health,info
management.info.env.enabled=true
```

---

### **Resolução 5**

**Propriedades em `application.properties`:**
```properties
info.app.name=Sistema de Gestao Academica
info.app.version=2.1.0
info.app.developer=Lucas
```

---

### **Resolução 6**

**1. Explicação:**
Ao adicionar o `spring-boot-starter-security`, todas as rotas/endpoints REST da aplicação passam a ser automaticamente protegidas por autenticação HTTP Basic. Requisições não autenticadas receberão o status HTTP `401 Unauthorized`.

**2. Configuração em `application.properties`:**
```properties
spring.security.user.name=admin
spring.security.user.password=senhaSegura123
```

---

### **Resolução 7**

As duas configurações necessárias no IntelliJ IDEA são:
1. **Settings / Preferences** $\rightarrow$ **Build, Execution, Deployment** $\rightarrow$ **Compiler**:
   - Marcar a caixa **`Build project automatically`**.
2. **Settings / Preferences** $\rightarrow$ **Advanced Settings**:
   - Marcar a caixa **`Allow auto-make to start even if developed application is currently running`**.

---

### **Resolução 8**

**Configurações em `application.properties`:**
```properties
logging.level.org.springframework=DEBUG
logging.level.com.escola.demo=TRACE
logging.file.name=escola-debug.log
```

---

### **Resolução 9**

**1. Gerar o arquivo JAR:**
```bash
./mvnw package
```
*(No Windows PowerShell / CMD: `mvnw package`)*

**2. Executar o JAR gerado:**
```bash
java -jar target/demo-0.0.1-SNAPSHOT.jar
```

**3. Executar diretamente sem empacotar manualmente:**
```bash
./mvnw spring-boot:run
```
*(No Windows: `mvnw spring-boot:run`)*

---

### **Resolução 10**

**1. Causa do erro:**
A anotação `@SpringBootApplication` inclui o `@ComponentScan`, que por padrão varre o pacote onde a classe principal está e **apenas os seus subpacotes**. Como a classe principal estava no pacote `com.escola.demo.controller.main`, ela só procurava por Beans dentro de `.main`. Os Controllers no pacote `com.escola.demo.rest` foram ignorados e não foram registrados no container Spring.

**2. Correção (Posicionamento e Código):**
A classe principal deve ser colocada no **pacote raiz** do projeto (`com.escola.demo`).

```java
package com.escola.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```
