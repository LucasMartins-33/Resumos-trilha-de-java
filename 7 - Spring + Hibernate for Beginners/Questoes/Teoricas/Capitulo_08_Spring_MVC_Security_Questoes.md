# Questões Teóricas - Capítulo 08: Spring MVC Security

### Questão 1
Qual é a principal diferença de comportamento do Spring Security entre uma aplicação REST API e uma aplicação Web Spring MVC com Thymeleaf?

<details>
<summary>👀 Ver Resposta</summary>

Aplicações REST API utilizam comunicação *stateless* (sem estado de sessão) e enviam credenciais/tokens a cada requisição HTTP. Aplicações Spring MVC utilizam comunicação *stateful* baseada em sessão de navegador (cookies `JSESSIONID`), exibem formulários HTML de login e realizam redirecionamentos de página pós-autenticação.
</details>

---

### Questão 2
Como registrar uma página de login HTML customizada no Spring Security através do `SecurityFilterChain`?

<details>
<summary>👀 Ver Resposta</summary>

Utiliza-se a configuração do `.formLogin()`:
```java
http.formLogin(form -> form
    .loginPage("/showMyLoginPage")
    .loginProcessingUrl("/authenticateTheUser")
    .permitAll()
);
```
</details>

---

### Questão 3
Quais são os nomes exatos de campos exigidos nos inputs de um formulário de login HTML para que o processamento do Spring Security funcione automaticamente?

<details>
<summary>👀 Ver Resposta</summary>

Os campos `<input>` do formulário HTML **devem ter obrigatoriamente** `name="username"` para o usuário e `name="password"` para a senha. O envio do formulário deve ser feito via método `POST` apontando para a URL configurada no `loginProcessingUrl()`.
</details>

---

### Questão 4
Como capturar e exibir mensagens de erro de autenticação e mensagens de logout no template HTML Thymeleaf?

<details>
<summary>👀 Ver Resposta</summary>

O Spring Security insere automaticamente parâmetros na URL em caso de falha (`?error`) ou saída (`?logout`). No Thymeleaf, eles são checados via:
* `<div th:if="${param.error}" class="alert alert-danger">Usuário ou senha inválidos.</div>`
* `<div th:if="${param.logout}" class="alert alert-success">Você foi desconectado.</div>`
</details>

---

### Questão 5
Por que o encerramento de sessão (Logout) em aplicações Web com Spring Security deve ser feito via requisição HTTP `POST`?

<details>
<summary>👀 Ver Resposta</summary>

Para proteger a aplicação contra ataques de **CSRF (Cross-Site Request Forgery)** e evitar que o logout seja acionado acidentalmente por links estáticos `GET`, robôs de busca ou tags de imagem maliciosas.
</details>

---

### Questão 6
Para que serve a biblioteca `thymeleaf-extras-springsecurity6` em projetos Spring Boot?

<details>
<summary>👀 Ver Resposta</summary>

Ela fornece a namespace `sec:*` para uso direto nos arquivos HTML/Thymeleaf. Permite inspecionar dados do usuário logado (como `sec:authentication="principal.username"`) e controlar a exibição condicional de componentes baseando-se em papéis/roles (como `sec:authorize="hasRole('ADMIN')"`).
</details>

---

### Questão 7
O que acontece com o código HTML contido dentro de um elemento anotado com `sec:authorize="hasRole('ADMIN')"` caso o usuário logado seja um funcionário comum (`EMPLOYEE`)?

<details>
<summary>👀 Ver Resposta</summary>

O bloco HTML **não é renderizado pelo servidor**. O Spring Security e o Thymeleaf omitem completamente o código do HTML final gerado, impedindo que o conteúdo ou links protegidos fiquem visíveis no código-fonte do navegador (*Inspect Element*).
</details>

---

### Questão 8
Como mapear uma página customizada para o erro **403 Forbidden (Acesso Negado)** no Spring Security?

<details>
<summary>👀 Ver Resposta</summary>

Mapeia-se a página no `SecurityFilterChain` usando o tratamento de exceções:
```java
http.exceptionHandling(configurer -> 
    configurer.accessDeniedPage("/access-denied")
);
```
E cria-se um método `@GetMapping("/access-denied")` no controller que direciona para o template HTML correspondente.
</details>

---

### Questão 9
Como habilitar o suporte para autenticação de usuários via banco de dados MySQL no Spring Security usando o `JdbcUserDetailsManager`?

<details>
<summary>👀 Ver Resposta</summary>

Declara-se um Bean de `UserDetailsManager` injetando o `DataSource` da aplicação:
```java
@Bean
public UserDetailsManager userDetailsManager(DataSource dataSource) {
    return new JdbcUserDetailsManager(dataSource);
}
```
</details>

---

### Questão 10
Qual é a restrição de tamanho da coluna de senha no banco de dados quando se utiliza codificação de senhas com o algoritmo BCrypt?

<details>
<summary>👀 Ver Resposta</summary>

A coluna deve ser criada no banco com no mínimo `VARCHAR(68)` para comportar os 60 caracteres de hash do BCrypt mais o prefixo de 8 caracteres do algoritmo (`{bcrypt}`). Tamanhos menores truncarão o hash e impedirão a autenticação dos usuários.
</details>
