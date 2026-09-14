# Capítulo 11: What's New in Spring Boot 4

Este resumo consolida as novidades, melhorias, arquitetura, novidades de versionamento de APIs, migrações e atualizações do ecossistema introduzidos com o **Spring Boot 4** (lançado no final de 2025, baseado no **Spring Framework 7**).

---

## 📌 1. Visão Geral e Filosofia do Spring Boot 4

O Spring Boot 4 é construído sobre o **Spring Framework 7** e foi desenvolvido em torno de 4 pilares principais:

1. **Faster (Mais Rápido)**: Execução otimizada em versões modernas do Java (Java 17+ baseline, recomendados Java 21 ou 25), tempo de inicialização (*startup time*) reduzido e menor uso de memória RAM.
2. **Leaner (Mais Enxuto)**: Arquitetura altamente modularizada para diminuir o tamanho do *classpath* e eliminar dependências desnecessárias.
3. **Safer (Mais Seguro)**: Integração nativa com **JSpecify** para detecção de inconsistências de valores `null` em tempo de compilação (*Null Safety*) e atualizações de segurança runtime.
4. **Cloud Readiness & Observability**: Probes automáticos do Kubernetes configurados por padrão (`/actuator/health/liveness` e `/actuator/health/readiness`) e observabilidade aprimorada.

---

## 📌 2. Comparativo de Migração: Spring Boot 2 ➔ 3 vs Spring Boot 3 ➔ 4

> [!IMPORTANT]
> **Suavidade de Atualização no Spring Boot 4**:
> Ao contrário da dolorosa migração do Spring Boot 2 para 3 (onde o pacote `javax.*` foi massivamente renomeado para `jakarta.*`, impactando ~50% do código dos projetos), a transição do **Spring Boot 3 para o Spring Boot 4 é incremental e com pouquíssimas *breaking changes***.

### Alinhamento do Ecossistema:
* **Java**: Mínimo Java 17 (suporte nativo recomendado para Java 21 / Java 25 LTS).
* **Hibernate**: Atualizado para **Hibernate 7**.
* **Jackson**: Atualizado para **Jackson 3**.
* **Tomcat**: Atualizado para **Tomcat 11**.
* **JUnit**: Atualizado para **JUnit 6**.

---

## 📌 3. Principais Novidades Técnicas do Spring Boot 4

### 3.1 Versionamento Nativo de REST APIs (First-Class API Versioning)
No Spring Boot 4, o versionamento de APIs REST é um recurso nativo do framework. Não é mais necessário criar *Filters*, *interceptors* customizados ou métodos duplicados com URLs hardcoded.

#### Configuração via `application.properties`:
```properties
# Define que o número da versão será extraído do segmento de índice 1 no caminho do recurso
spring.mvc.api-version.use-path-segment=1
```
*(No caminho `/api/v1/employees`, o índice 0 é `api` e o índice 1 é `v1`. O Spring Boot faz o parse automático, ignora o prefixo "v" e identifica a versão 1).*

#### Implementação no Controller (`@RestController`):
```java
package com.luv2code.springboot.demo.rest;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/{version}/hello") // Placeholder flexível para a versão no path
public class HelloRestController {

    @GetMapping(version = "1")
    public String getHelloV1() {
        return "Hello World from API Version 1!";
    }

    @GetMapping(version = "2")
    public String getHelloV2() {
        return "Hello World from API Version 2!";
    }

    @GetMapping(version = "3")
    public String getHelloV3() {
        return "Hello World from API Version 3!";
    }
}
```

---

### 3.2 Jackson 3 como Biblioteca JSON Padrão & Alterações de Importação

O Spring Boot 4 adota o **Jackson 3** como seu parseador JSON padrão.

#### Mudança Importante nos Pacotes (Renomeação):
* **Jackson 2 (Spring Boot 3)**: `com.fasterxml.jackson.databind.*`
* **Jackson 3 (Spring Boot 4)**: `tools.jackson.databind.*`

> [!NOTE]
> Essa mudança só afeta diretamente o seu código Java caso você instancie/manipule classes do Jackson manualmente (como em métodos `PATCH` REST usando `JsonMapper`).

#### Substituição de `ObjectMapper` por `JsonMapper`:
No Jackson 3 / Spring Boot 4, a API preferencial para manipulação de JSON é a interface **`JsonMapper`**, autoconfigurada automaticamente como Bean pelo Spring:

```java
@RestController
@RequestMapping("/api")
public class EmployeeRestController {

    private final JsonMapper jsonMapper; // Injetado automaticamente pelo Spring Boot 4

    public EmployeeRestController(JsonMapper jsonMapper) {
        this.jsonMapper = jsonMapper;
    }

    @PatchMapping("/employees/{employeeId}")
    public Employee patchEmployee(@PathVariable int employeeId, @RequestBody Map<String, Object> patchPayload) {
        Employee existingEmployee = employeeService.findById(employeeId);
        
        // Em Jackson 3, o JsonMapper não exige cláusula 'throws JsonMappingException' checada
        jsonMapper.updateValue(existingEmployee, patchPayload);
        
        return employeeService.save(existingEmployee);
    }
}
```

---

### 3.3 Null Safety Nativo com JSpecify

O Spring Boot 4 inclui internamente suporte às anotações da biblioteca **JSpecify** (`@Nullable`, `@NonNull`).
* Permite que a IDE e ferramentas de análise estática (*linters*) identifiquem potenciais exceções do tipo `NullPointerException` antes da compilação.
* Seu uso no código da aplicação é **opcional** e não gera quebra de compatibilidade.

---

## 📌 4. Guia Prático de Migração do Spring Boot 3 para o Spring Boot 4

### Step 1: Atualizar a Versão no `pom.xml`
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.0.0</version> <!-- Ou versão 4.x mais recente -->
    <relativePath/>
</parent>
```

---

### Step 2: Renomear o Starter Web Depreciado
No Spring Boot 4, o starter `spring-boot-starter-web` foi depreciado e substituído por **`spring-boot-starter-web-mvc`**:

```xml
<!-- ANTES (Spring Boot 3) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- DEPOIS (Spring Boot 4) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web-mvc</artifactId>
</dependency>
```

---

### Step 3: Renomear o Starter de AOP / AspectJ
O starter antigo `spring-boot-starter-aop` foi atualizado para **`spring-boot-starter-aspectj`**:

```xml
<!-- ANTES (Spring Boot 3) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>

<!-- DEPOIS (Spring Boot 4) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aspectj</artifactId>
</dependency>
```

---

## 📋 Tabela Resumo: Principais Mudanças entre Versões

| Recurso / Componente | Spring Boot 3 | Spring Boot 4 |
| :--- | :--- | :--- |
| **Spring Framework Base** | Spring Framework 6 | **Spring Framework 7** |
| **Java Baseline** | Java 17 | **Java 17+** (Recomendado Java 21 ou 25) |
| **Starter Web** | `spring-boot-starter-web` | **`spring-boot-starter-web-mvc`** |
| **Starter AOP** | `spring-boot-starter-aop` | **`spring-boot-starter-aspectj`** |
| **Pacote Jackson** | `com.fasterxml.jackson.databind.*` | **`tools.jackson.databind.*`** |
| **Classe Preferencial JSON** | `ObjectMapper` | **`JsonMapper`** |
| **Versionamento de REST APIs** | Lógica manual ou filtros customizados | **Nativo** via `version = "1"` no `@GetMapping` / `@RequestMapping` |
| **Null Safety** | Anotações Spring (`@NonNull`) | **JSpecify** integrado nativamente |
