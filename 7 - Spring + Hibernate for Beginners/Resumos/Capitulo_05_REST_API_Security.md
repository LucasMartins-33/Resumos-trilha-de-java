# Capítulo 05: REST API Security

Este resumo abrange a segurança em APIs RESTful com **Spring Boot 4** e **Spring Security**. Ele aborda os conceitos fundamentais de Autenticação e Autorização, configuração de usuários em memória (`InMemoryUserDetailsManager`), restrição de URLs e métodos HTTP por papéis/roles (`SecurityFilterChain`), prevenção de ataques CSRF, e autenticação via **JDBC** (banco de dados) com armazenamento de senhas em texto puro, com criptografia **BCrypt** e usando tabelas/colunas customizadas.

---

## 📌 1. Visão Geral do Spring Security

O Spring Security é o arcabouço padrão para proteção de aplicações Spring, operando através de uma cadeia de **Servlet Filters** (*Filter Chain*) que interceptam requisições HTTP antes de atingirem os controllers.

```text
[Cliente REST / Browser]
          │
          ▼ (Requisição HTTP)
┌───────────────────────────────────────────────────────────┐
│ Spring Security Filter Chain                              │
│  ├─ 1. Autenticação (Verifica Usuário e Senha)            │
│  └─ 2. Autorização (Verifica Roles/Permissões na URL)     │
└───────────────────────────────────────────────────────────┘
          │ (Permitido)
          ▼
┌───────────────────────────────────────────────────────────┐
│ Controlador REST / Recurso Protegido                       │
└───────────────────────────────────────────────────────────┘
```

### Dois Pilares Fundamentais
1. **Autenticação (`Authentication`)**: Valida "quem é você" (confirma se o ID de usuário e a senha são válidos).
2. **Autorização (`Authorization`)**: Valida "o que você pode fazer" (confirma se o usuário autenticado possui o papel/role exigido para acessar aquele endpoint específico).

---

## 📌 2. Habilitando a Segurança Básica no Spring Boot

Ao adicionar a dependência do Spring Security no `pom.xml`, todas as rotas da aplicação passam a ser protegidas automaticamente via **HTTP Basic**:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### Configuração Padrão Inicial
* **Usuário Padrão**: `user`
* **Senha Padrão**: Gerada aleatoriamente a cada inicialização e impressa no log do console (`Using generated security password: ...`).

### Alterando Usuário e Senha Rápidos no `application.properties`
*(Útil apenas para desenvolvimento simples/testes rápidos)*
```properties
spring.security.user.name=scott
spring.security.user.password=test123
```

---

## 📌 3. Autenticação em Memória (`InMemoryUserDetailsManager`)

Configuração de usuários e papéis diretamente via código Java usando `@Configuration` e a definição de um Bean de `UserDetailsManager`.

### Formato de Senhas no Spring Security
O Spring Security exige que o hash da senha indique o algoritmo de codificação em colchetes angulares `{id}`:
* `{noop}`: Indica senha em texto puro (*no-operation*, sem criptografia/hashing). **Não recomendado para produção!**
* `{bcrypt}`: Indica senha criptografada com o algoritmo BCrypt.

### Código Java para Usuários em Memória (`DemoSecurityConfig.java`)

```java
package com.luv2code.springboot.cruddemo.security;

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
                .password("{noop}test123") // Texto puro
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

## 📌 4. Autorização Granular e Restrição de URLs (`SecurityFilterChain`)

Podemos restringir os endpoints da API por método HTTP (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`) e pelas Roles do usuário utilizando a classe de configuração `SecurityFilterChain`.

### Matriz de Permissões da API de Funcionários (`/api/employees`)

| Método HTTP | Endpoint | Roles Permitidas | Descrição |
| :--- | :--- | :--- | :--- |
| **`GET`** | `/api/employees`, `/api/employees/**` | `EMPLOYEE`, `MANAGER`, `ADMIN` | Leitura de todos ou de um funcionário. |
| **`POST`** | `/api/employees` | `MANAGER`, `ADMIN` | Criação de novo funcionário. |
| **`PUT`** | `/api/employees`, `/api/employees/**` | `MANAGER`, `ADMIN` | Atualização completa de funcionário. |
| **`PATCH`** | `/api/employees/**` | `MANAGER`, `ADMIN` | Atualização parcial de funcionário. |
| **`DELETE`**| `/api/employees/**` | `ADMIN` | Exclusão de funcionário. |

> [!IMPORTANT]
> **Desativação de CSRF em APIs REST**:
> Para APIs REST **stateless** (sem sessão baseada em navegador), a proteção contra **CSRF (Cross-Site Request Forgery)** deve ser desativada com `.csrf(csrf -> csrf.disable())`, conforme recomendação oficial da documentação Spring.

### Código Java de Configuração do `SecurityFilterChain`

```java
package com.luv2code.springboot.cruddemo.security;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.Customizer;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class DemoSecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {

        http.authorizeHttpRequests(configurer ->
                configurer
                        // 1. LEITURA (GET): Aberto para EMPLOYEE, MANAGER e ADMIN
                        .requestMatchers(HttpMethod.GET, "/api/employees").hasRole("EMPLOYEE")
                        .requestMatchers(HttpMethod.GET, "/api/employees/**").hasRole("EMPLOYEE")
                        // 2. CRIAÇÃO (POST): Requer MANAGER ou ADMIN
                        .requestMatchers(HttpMethod.POST, "/api/employees").hasRole("MANAGER")
                        // 3. ATUALIZAÇÃO COMPLETA (PUT): Requer MANAGER ou ADMIN
                        .requestMatchers(HttpMethod.PUT, "/api/employees").hasRole("MANAGER")
                        .requestMatchers(HttpMethod.PUT, "/api/employees/**").hasRole("MANAGER")
                        // 4. ATUALIZAÇÃO PARCIAL (PATCH): Requer MANAGER ou ADMIN
                        .requestMatchers(HttpMethod.PATCH, "/api/employees/**").hasRole("MANAGER")
                        // 5. EXCLUSÃO (DELETE): Exclusivo para ADMIN
                        .requestMatchers(HttpMethod.DELETE, "/api/employees/**").hasRole("ADMIN")
        );

        // Habilita a autenticação HTTP Basic
        http.httpBasic(Customizer.withDefaults());

        // Desativa a proteção CSRF para APIs REST
        http.csrf(csrf -> csrf.disable());

        return http.build();
    }
}
```

> [!NOTE]
> **Sintaxe do Wildcard `**`**: O uso de `/**` (ex: `/api/employees/**`) corresponde a qualquer subcaminho ou ID passado na URL (ex: `/api/employees/1`).

---

## 📌 5. Autenticação via Banco de Dados (JDBC Authentication)

Em aplicações reais, usuários, senhas e papéis são armazenados em tabelas no banco de dados.

### 5.1 Esquema de Tabelas Padrão do Spring Security
O Spring Security possui um esquema relacional pré-definido de duas tabelas: **`users`** e **`authorities`**.

#### Script SQL para Criação das Tabelas Padrão:
```sql
USE `employee_directory`;

DROP TABLE IF EXISTS `authorities`;
DROP TABLE IF EXISTS `users`;

-- Tabela de Usuários
CREATE TABLE `users` (
  `username` varchar(50) NOT NULL,
  `password` varchar(68) NOT NULL, -- Mínimo de 68 caracteres para suportar BCrypt!
  `enabled` tinyint NOT NULL,
  PRIMARY KEY (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

-- Tabela de Permissões/Roles
CREATE TABLE `authorities` (
  `username` varchar(50) NOT NULL,
  `authority` varchar(50) NOT NULL,
  UNIQUE KEY `authorities_idx_1` (`username`,`authority`),
  CONSTRAINT `authorities_ibfk_1` FOREIGN KEY (`username`) REFERENCES `users` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
```

#### Regra Importante sobre os Nomes de Roles
No banco de dados, o Spring Security exige por padrão que os nomes das roles na tabela `authorities` tenham o prefixo **`ROLE_`**:
* Exemplo: `ROLE_EMPLOYEE`, `ROLE_MANAGER`, `ROLE_ADMIN`.

---

### 5.2 Autenticação JDBC com Senhas em Texto Puro (`{noop}`)

#### Script SQL de Inserção:
```sql
INSERT INTO `users` VALUES 
('john', '{noop}test123', 1),
('mary', '{noop}test123', 1),
('susan', '{noop}test123', 1);

INSERT INTO `authorities` VALUES 
('john', 'ROLE_EMPLOYEE'),
('mary', 'ROLE_EMPLOYEE'),
('mary', 'ROLE_MANAGER'),
('susan', 'ROLE_EMPLOYEE'),
('susan', 'ROLE_MANAGER'),
('susan', 'ROLE_ADMIN');
```

#### Código Java de Configuração (`JdbcUserDetailsManager`):
Ao usar as tabelas e colunas padrão, não é necessário escrever nenhuma query SQL manual.

```java
@Bean
public UserDetailsManager userDetailsManager(DataSource dataSource) {
    // Injeta automaticamente o DataSource do MySQL e busca as tabelas padrão "users" e "authorities"
    return new JdbcUserDetailsManager(dataSource);
}
```

---

### 5.3 Autenticação JDBC com Criptografia BCrypt (`{bcrypt}`)

O **BCrypt** é um algoritmo de hashing unidirecional seguro que inclui *salting* aleatório e proteção contra ataques de força bruta.

> [!WARNING]
> **Tamanho Obrigatório da Coluna `password`**:
> O hash do BCrypt possui 60 caracteres. Somado aos 8 caracteres do identificador `{bcrypt}`, a coluna `password` da tabela do banco de dados deve ter **tamanho de no mínimo 68 caracteres** (`VARCHAR(68)`).

#### Script SQL de Inserção com Hash BCrypt:
```sql
-- Senha plana original: "fun123"
INSERT INTO `users` VALUES 
('john', '{bcrypt}$2a$10$eSkriC56nB8n.1i9j/X0v.7Pq/iWp3zJ9V54hP6ZkO9o8U8wZJ9i2', 1),
('mary', '{bcrypt}$2a$10$eSkriC56nB8n.1i9j/X0v.7Pq/iWp3zJ9V54hP6ZkO9o8U8wZJ9i2', 1),
('susan', '{bcrypt}$2a$10$eSkriC56nB8n.1i9j/X0v.7Pq/iWp3zJ9V54hP6ZkO9o8U8wZJ9i2', 1);
```

---

## 📌 6. Autenticação JDBC com Tabelas e Colunas Customizadas

Se a empresa utilizar um esquema de banco de dados legado com nomes de tabelas e colunas diferentes do padrão do Spring Security (ex: tabelas `members` e `roles`), basta fornecer as consultas SQL de busca ao `JdbcUserDetailsManager`.

### Esquema Customizado no Banco de Dados
* Tabela de Usuários: **`members`** (Colunas: `user_id`, `pw`, `active`)
* Tabela de Roles: **`roles`** (Colunas: `user_id`, `role`)

#### Script SQL para Tabelas Customizadas:
```sql
DROP TABLE IF EXISTS `roles`;
DROP TABLE IF EXISTS `members`;

CREATE TABLE `members` (
  `user_id` varchar(50) NOT NULL,
  `pw` char(68) NOT NULL,
  `active` tinyint NOT NULL,
  PRIMARY KEY (`user_id`)
);

CREATE TABLE `roles` (
  `user_id` varchar(50) NOT NULL,
  `role` varchar(50) NOT NULL,
  UNIQUE KEY `roles_idx_1` (`user_id`,`role`),
  CONSTRAINT `roles_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `members` (`user_id`)
);

INSERT INTO `members` VALUES 
('john', '{bcrypt}$2a$10$eSkriC56nB8n.1i9j/X0v.7Pq/iWp3zJ9V54hP6ZkO9o8U8wZJ9i2', 1),
('mary', '{bcrypt}$2a$10$eSkriC56nB8n.1i9j/X0v.7Pq/iWp3zJ9V54hP6ZkO9o8U8wZJ9i2', 1),
('susan', '{bcrypt}$2a$10$eSkriC56nB8n.1i9j/X0v.7Pq/iWp3zJ9V54hP6ZkO9o8U8wZJ9i2', 1);

INSERT INTO `roles` VALUES 
('john', 'ROLE_EMPLOYEE'),
('mary', 'ROLE_EMPLOYEE'),
('mary', 'ROLE_MANAGER'),
('susan', 'ROLE_EMPLOYEE'),
('susan', 'ROLE_MANAGER'),
('susan', 'ROLE_ADMIN');
```

#### Código Java com Queries Personalizadas (`DemoSecurityConfig.java`):

```java
package com.luv2code.springboot.cruddemo.security;

import javax.sql.DataSource;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.provisioning.JdbcUserDetailsManager;
import org.springframework.security.provisioning.UserDetailsManager;

@Configuration
public class DemoSecurityConfig {

    @Bean
    public UserDetailsManager userDetailsManager(DataSource dataSource) {

        JdbcUserDetailsManager jdbcUserDetailsManager = new JdbcUserDetailsManager(dataSource);

        // 1. Query para buscar o usuário pelo id (Retorna: username, password, enabled)
        jdbcUserDetailsManager.setUsersByUsernameQuery(
                "select user_id, pw, active from members where user_id=?"
        );

        // 2. Query para buscar as roles do usuário (Retorna: username, role)
        jdbcUserDetailsManager.setAuthoritiesByUsernameQuery(
                "select user_id, role from roles where user_id=?"
        );

        return jdbcUserDetailsManager;
    }
}
```

---

## 📌 7. Habilitando Log de Depuração do Spring Security

Caso ocorram erros de autenticação (ex: Erro `401 Unauthorized` ou `403 Forbidden`) ou nomes incorretos de colunas/queries, é útil ativar os logs de depuração no `application.properties`:

```properties
logging.level.org.springframework.security=DEBUG
```

---

## 📋 Tabela Resumo dos Métodos e Classes do Capítulo 05

| Classe / Interface / Método | Pacote | Descrição |
| :--- | :--- | :--- |
| **`SecurityFilterChain`** | `...security.web` | Define a cadeia de filtros e regras de segurança HTTP. |
| **`InMemoryUserDetailsManager`**| `...security.provisioning` | Implementação de gerenciador de usuários mantidos em memória. |
| **`JdbcUserDetailsManager`** | `...security.provisioning` | Implementação de gerenciador de usuários integrados ao banco de dados via JDBC. |
| **`requestMatchers(method, url)`**| `...configurer` | Mapeia regras de segurança para combinações de método HTTP e caminhos de URL. |
| **`hasRole("ROLE")`** | `...configurer` | Exige que o usuário autenticado possua a Role especificada. |
| **`httpBasic()`** | `...configurer` | Habilita a autenticação no padrão HTTP Basic. |
| **`csrf().disable()`** | `...configurer` | Desativa a proteção CSRF para APIs REST stateless. |
| **`setUsersByUsernameQuery()`** | `...JdbcUserDetailsManager` | Define a consulta SQL personalizada para busca de usuários. |
| **`setAuthoritiesByUsernameQuery()`**| `...JdbcUserDetailsManager` | Define a consulta SQL personalizada para busca de permissões/roles. |
