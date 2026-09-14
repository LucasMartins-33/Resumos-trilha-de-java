# Capítulo 08: Spring MVC Security

Este resumo aborda a integração do **Spring Security** em aplicações **Spring MVC (Server-Side com Thymeleaf)** no **Spring Boot 4**. Ele cobre a personalização do formulário de login com Bootstrap, mensagens de erro e logout, exibição de nome de usuário e papéis na interface, restrição de acesso por rotas e renderização condicional de conteúdo no HTML (`thymeleaf-extras-springsecurity6`), tratamento de acesso negado (Erro 403), além de autenticação JDBC em banco de dados MySQL com BCrypt e tabelas customizadas.

---

## 📌 1. Visão Geral do Spring Security no Spring MVC

Diferente de APIs REST *stateless*, aplicações Web Spring MVC gerenciam sessão de usuário no navegador (cookies de sessão `JSESSIONID`). O Spring Security atua como uma cadeia de **Servlet Filters** interceptando todas as requisições HTTP antes de chegarem aos controllers MVC.

### Dependências Maven (`pom.xml`)
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<!-- Suporte para tags do Spring Security dentro de templates Thymeleaf -->
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity6</artifactId>
</dependency>
```

---

## 📌 2. Configuração Básica e Usuários em Memória (`InMemoryUserDetailsManager`)

Definição de permissões e usuários em memória para ambientes de testes.

### `DemoSecurityConfig.java`
```java
package com.luv2code.springboot.demosecurity.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

@Configuration
public class DemoSecurityConfig {

    @Bean
    public InMemoryUserDetailsManager userDetailsManager() {
        UserDetails john = User.builder()
                .username("john")
                .password("{noop}test123")
                .roles("EMPLOYEE")
                .build();

        UserDetails mary = User.builder()
                .username("mary")
                .password("{noop}test123")
                .roles("EMPLOYEE", "MANAGER")
                .build();

        UserDetails susan = User.builder()
                .username("susan")
                .password("{noop}test123")
                .roles("EMPLOYEE", "MANAGER", "ADMIN")
                .build();

        return new InMemoryUserDetailsManager(john, mary, susan);
    }
}
```

---

## 📌 3. Customização do Formulário de Login & Logout com Bootstrap 5

Para evitar o formulário de login padrão simples do Spring, podemos criar uma página HTML customizada com Bootstrap e registrá-la no `SecurityFilterChain`.

### 3.1 Registrando o Login Customizado e Logout no Spring Security

```java
package com.luv2code.springboot.demosecurity.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class DemoSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http.authorizeHttpRequests(configurer ->
                configurer
                        .requestMatchers("/").hasRole("EMPLOYEE")
                        .requestMatchers("/leaders/**").hasRole("MANAGER")
                        .requestMatchers("/systems/**").hasRole("ADMIN")
                        .anyRequest().authenticated()
        )
        .formLogin(form ->
                form
                        // URL para onde o Spring vai redirecionar ao pedir login
                        .loginPage("/showMyLoginPage")
                        // URL para onde o formulário HTML envia os dados (POST)
                        .loginProcessingUrl("/authenticateTheUser")
                        // Permite acesso público à tela de login
                        .permitAll()
        )
        .logout(logout ->
                logout
                        // Habilita logout padrão no endpoint /logout
                        .permitAll()
        )
        .exceptionHandling(configurer ->
                configurer
                        // Página customizada para Erro 403 (Acesso Negado)
                        .accessDeniedPage("/access-denied")
        );

        return http.build();
    }
}
```

---

### 3.2 O Controller de Autenticação (`LoginController.java`)

O Spring Security gerencia automaticamente a rota `/authenticateTheUser` (não é necessário criar método para ela). É necessário criar apenas os métodos de exibição das telas (`GET`):

```java
package com.luv2code.springboot.demosecurity.controller;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class LoginController {

    @GetMapping("/showMyLoginPage")
    public String showMyLoginPage() {
        return "fancy-login"; // Retorna fancy-login.html em src/main/resources/templates/
    }

    @GetMapping("/access-denied")
    public String showAccessDenied() {
        return "access-denied"; // Retorna access-denied.html
    }
}
```

---

### 3.3 O Template HTML do Login (`fancy-login.html`)

O Spring Security passa automaticamente parâmetros na URL para informar falha de autenticação (`?error`) ou logout bem-sucedido (`?logout`).

> [!IMPORTANT]
> **Nomes Padrão de Campos**:
> Os inputs do formulário HTML **devem** ter exatamente `name="username"` e `name="password"` para que o Spring Security leia as credenciais corretamente.

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title>Custom Login Page</title>
    <meta charset="utf-8">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
</head>
<body>

<div class="container mt-5" style="max-width: 400px;">
    <h3 class="mb-3 text-center">Login</h3>

    <!-- Mensagem de Erro de Autenticação (?error) -->
    <div th:if="${param.error}" class="alert alert-danger">
        Invalid username and password.
    </div>

    <!-- Mensagem de Logout Sucesso (?logout) -->
    <div th:if="${param.logout}" class="alert alert-success">
        You have been logged out.
    </div>

    <!-- Formulário que submete para /authenticateTheUser via POST -->
    <form th:action="@{/authenticateTheUser}" method="POST">

        <div class="mb-3">
            <input type="text" name="username" class="form-control" placeholder="Username" required />
        </div>

        <div class="mb-3">
            <input type="password" name="password" class="form-control" placeholder="Password" required />
        </div>

        <button type="submit" class="btn btn-primary w-100">Login</button>
    </form>
</div>

</body>
</html>
```

---

### 3.4 Formulário de Logout no HTML
Diferente de REST, em aplicações web o logout deve ser um envio por método `POST` (geralmente encapsulado num formulário) para evitar ataques de CSRF ou logouts não intencionais:

```html
<form th:action="@{/logout}" method="POST">
    <input type="submit" value="Logout" class="btn btn-outline-danger btn-sm" />
</form>
```

---

## 📌 4. Exibição de Dados do Usuário e Controle de Visibilidade com Thymeleaf

Com o pacote `thymeleaf-extras-springsecurity6`, podemos acessar informações da sessão do usuário e esconder/exibir elementos do HTML com base nas roles diretamente no template.

### Adicionando a Namespace no HTML:
```html
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/extras/spring-security">
```

### Exibindo Nome do Usuário e Roles no HTML:
```html
<!-- Exibe o Username do usuário logado -->
User: <span sec:authentication="principal.username"></span><br/>

<!-- Exibe a Lista de Roles (Authorities) -->
Roles: <span sec:authentication="principal.authorities"></span>
```

### Ocultando/Exibindo Elementos Condicionalmente (`sec:authorize`):

```html
<!-- Conteúdo visível APENAS para quem possui a role MANAGER -->
<div sec:authorize="hasRole('MANAGER')" class="alert alert-info">
    <a th:href="@{/leaders}">Leadership Retreat Info (Managers Only)</a>
</div>

<!-- Conteúdo visível APENAS para quem possui a role ADMIN -->
<div sec:authorize="hasRole('ADMIN')" class="alert alert-warning">
    <a th:href="@{/systems}">IT Systems Admin Panel (Admins Only)</a>
</div>
```

> [!TIP]
> **Segurança de Código**: O comando `sec:authorize` impede totalmente a renderização do bloco HTML no servidor. O conteúdo omitido **não aparece no código-fonte enviado ao navegador**.

---

## 📌 5. Autenticação JDBC e Banco de Dados MySQL

### 5.1 Configurando o `application.properties`
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/employee_directory
spring.datasource.username=springstudent
spring.datasource.password=springstudent

# Opcional para Debug de Autenticação em Desenvolvimento
logging.level.org.springframework.security=DEBUG
```

---

### 5.2 Estrutura com Criptografia BCrypt (`DemoSecurityConfig.java`)

Injeção do `DataSource` gerenciado pelo Spring Boot para consulta direta às tabelas de autenticação.

```java
@Bean
public UserDetailsManager userDetailsManager(DataSource dataSource) {
    return new JdbcUserDetailsManager(dataSource);
}
```

#### Requisitos do Banco de Dados:
* Tabela **`users`**: Coluna `password` criada com `VARCHAR(68)` para suportar o formato `{bcrypt}...` (60 caracteres de hash + 8 caracteres da tag de algoritmo).
* Tabela **`authorities`**: Roles com o prefixo obrigatorio `ROLE_` (ex: `ROLE_EMPLOYEE`, `ROLE_MANAGER`).

---

### 5.3 Suporte para Tabelas Customizadas (`members` e `roles`)

Se o banco de dados da empresa possuir nomes de tabelas ou colunas diferentes, passamos as queries personalizadas via métodos setter:

```java
@Bean
public UserDetailsManager userDetailsManager(DataSource dataSource) {

    JdbcUserDetailsManager jdbcUserDetailsManager = new JdbcUserDetailsManager(dataSource);

    // Consulta customizada de usuários: (username, password, enabled)
    jdbcUserDetailsManager.setUsersByUsernameQuery(
            "select user_id, pw, active from members where user_id=?"
    );

    // Consulta customizada de permissões: (username, authority)
    jdbcUserDetailsManager.setAuthoritiesByUsernameQuery(
            "select user_id, role from roles where user_id=?"
    );

    return jdbcUserDetailsManager;
}
```

---

## 📋 Tabela Resumo das Anotações e Tags do Capítulo 08

| Anotação / Tag HTML | Origem | Propósito / Descrição |
| :--- | :--- | :--- |
| **`loginPage("/url")`** | `HttpSecurity` Java | Define a rota customizada da tela de login. |
| **`loginProcessingUrl("/url")`**| `HttpSecurity` Java | Define o endpoint onde o Spring Security intercepta e autentica os formulários POST. |
| **`accessDeniedPage("/url")`** | `HttpSecurity` Java | Mapeia o redirecionamento para a tela personalizada de erro de autorização 403. |
| **`sec:authentication="principal.username"`** | Thymeleaf Security Tag | Insere o nome do usuário atualmente autenticado no nó HTML. |
| **`sec:authentication="principal.authorities"`** | Thymeleaf Security Tag | Insere a lista de autoridades/roles do usuário no nó HTML. |
| **`sec:authorize="hasRole('ROLE')"`** | Thymeleaf Security Tag | Renderiza o bloco HTML condicionalmente apenas se o usuário possuir a Role exigida. |
| **`${param.error}`** | Thymeleaf Expression | Verifica a presença do parâmetro de URL indicando falha de login. |
| **`${param.logout}`** | Thymeleaf Expression | Verifica a presença do parâmetro de URL indicando encerramento de sessão. |
