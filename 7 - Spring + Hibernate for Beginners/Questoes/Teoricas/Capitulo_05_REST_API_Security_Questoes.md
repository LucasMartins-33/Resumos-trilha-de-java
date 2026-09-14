# Questões Teóricas - Capítulo 05: REST API Security

### Questão 1
Qual é a arquitetura fundamental do Spring Security para proteger requisições HTTP em uma aplicação Spring Boot?

> [!faq]- Resposta
> O Spring Security utiliza uma cadeia de **Servlet Filters (Filter Chain)** que intercepta todas as requisições HTTP que chegam à aplicação antes que elas atinjam os controladores REST. Se a requisição não for autenticada ou autorizada, o filtro interrompe a execução e retorna um código de erro HTTP (como 401 Unauthorized ou 403 Forbidden).

---

### Questão 2
Como declarar um Bean de `SecurityFilterChain` no Spring Security moderno usando Java Configuration?

> [!faq]- Resposta
> Declara-se um método anotado com `@Bean` em uma classe `@Configuration` retornando um `SecurityFilterChain` e recebendo o objeto `HttpSecurity`. Nele, configuram-se as regras de autorização via `.authorizeHttpRequests()`, o tipo de autenticação (ex: `.httpBasic(Customizer.withDefaults())`) e a desativação de proteção CSRF para APIs REST stateless via `.csrf(csrf -> csrf.disable())`.

---

### Questão 3
Por que a proteção CSRF (Cross-Site Request Forgery) é geralmente desativada em APIs REST stateless?

> [!faq]- Resposta
> A proteção CSRF é projetada para aplicações web tradicionais mantidas por cookies de sessão em navegadores. APIs REST *stateless* não utilizam sessões HTTP para autenticação (em geral utilizam HTTP Basic ou tokens JWT a cada requisição), tornando os ataques de CSRF irrelevantes e permitindo a desativação segura do mecanismo.

---

### Questão 4
O que é o `InMemoryUserDetailsManager` e para quais cenários ele é recomendado?

> [!faq]- Resposta
> É uma implementação simples do Spring Security que armazena os dados dos usuários, senhas e papéis (roles) diretamente na memória RAM durante a execução da aplicação. É recomendado apenas para ambientes de testes, protótipos ou provas de conceito (PoC).

---

### Questão 5
Qual é o papel da criptografia **BCrypt** no armazenamento de senhas e por que senhas em texto puro (`{noop}`) não devem ser usadas em produção?

> [!faq]- Resposta
> O BCrypt é uma função de hashing unidirecional adaptativa que incorpora um "sal" (salt) aleatório e um fator de custo computacional para proteger as senhas contra ataques de força bruta e tabelas rainbow. Senhas em texto puro (`{noop}`) ficam expostas caso o banco de dados seja comprometido, violando os princípios básicos de segurança.

---

### Questão 6
Como funciona a estrutura de tabelas e colunas padrão esperada pelo `JdbcUserDetailsManager` do Spring Security?

> [!faq]- Resposta
> O `JdbcUserDetailsManager` busca por duas tabelas padrão no banco de dados:
> 1. `users`: com as colunas `username` (PK), `password` (com o prefixo do algoritmo, ex: `{bcrypt}`) e `enabled` (booleano).
> 2. `authorities`: com as colunas `username` (FK) e `authority` (papel do usuário, obrigatoriamente prefixado com `ROLE_`, ex: `ROLE_EMPLOYEE`).

---

### Questão 7
Como configurar o `JdbcUserDetailsManager` para utilizar um esquema de banco de dados com nomes de tabelas e colunas customizadas?

> [!faq]- Resposta
> Instancia-se o `JdbcUserDetailsManager(dataSource)` e definem-se consultas SQL customizadas através dos métodos:
> * `setUsersByUsernameQuery("select user_id, pw, active from members where user_id=?")`
> * `setAuthoritiesByUsernameQuery("select user_id, role from roles where user_id=?")`

---

### Questão 8
Qual é a diferença entre Restrição por Papel (`hasRole`) e Restrição por Autoridade (`hasAuthority`) no Spring Security?

> [!faq]- Resposta
> * **`hasRole('ADMIN')`**: Espera que a permissão cadastrada no banco de dados possua o prefixo `ROLE_` (ex: `ROLE_ADMIN`), embora o parâmetro passado na checagem omita o prefixo.
> * **`hasAuthority('DELETE_PRIVILEGE')`**: Checa a string exata cadastrada no banco sem adicionar nenhum prefixo automaticamente.

---

### Questão 9
O que acontece se a coluna de senha em uma tabela de banco de dados MySQL for criada com um tamanho inferior a `VARCHAR(68)` ao utilizar criptografia BCrypt?

> [!faq]- Resposta
> O hash gerado pelo BCrypt possui exatamente 60 caracteres. Somado ao identificador do algoritmo no Spring Security (`{bcrypt}` de 8 caracteres), o texto total ocupa 68 caracteres. Se a coluna for menor, o Spring Security truncará o hash e o processo de verificação de senha falhará perpetuamente com erro 401.

---

### Questão 10
Como restringir rotas HTTP no `SecurityFilterChain` por método e papéis de usuário (ex: permitir `GET` para funcionários e `DELETE` apenas para administradores)?

> [!faq]- Resposta
> Utilizam-se os encadeamentos em `.authorizeHttpRequests()`:
> ```java
> configurer
>   .requestMatchers(HttpMethod.GET, "/api/employees/**").hasRole("EMPLOYEE")
>   .requestMatchers(HttpMethod.DELETE, "/api/employees/**").hasRole("ADMIN");
> ```
