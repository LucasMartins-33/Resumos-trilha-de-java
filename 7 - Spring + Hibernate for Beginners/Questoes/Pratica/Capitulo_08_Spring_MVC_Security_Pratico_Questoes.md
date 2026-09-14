# Questões Práticas – Capítulo 08: Spring MVC Security

## Exercício 8.1 – Configurar autenticação HTTP Basic para rotas MVC
**Cenário**
Proteja todas as rotas `/admin/**` usando HTTP Basic com usuário `admin` e senha `admin123`.

```java
package com.example.security;

import org.springframework.context.annotation.*;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.*;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class BasicSecurityConfig {

    @Bean // 🟢 Atividade: usuário em memória
    public UserDetailsService users() {
        UserDetails admin = User.withDefaultPasswordEncoder()
            .username("admin")
            .password("admin123")
            .roles("ADMIN")
            .build();
        return new InMemoryUserDetailsManager(admin);
    }

    @Bean // 🟢 Atividade: filtro de segurança
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().permitAll())
            .httpBasic(); // 🟢 Atividade: habilitar Basic Auth
        return http.build();
    }
}
```

---

## Exercício 8.2 – Form login com página customizada
**Cenário**
Crie um `login.html` (Thymeleaf) e configure Spring Security para usá‑lo.

```java
http
    .formLogin()
        .loginPage("/login") // 🟢 Atividade: página customizada
        .permitAll();
```

```html
<!-- src/main/resources/templates/login.html -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head><title>Login</title></head>
<body>
<form th:action="@{/login}" method="post">
    <div><label>Usuário: <input type="text" name="username"/></label></div>
    <div><label>Senha: <input type="password" name="password"/></label></div>
    <div><button type="submit">Entrar</button></div>
</form>
</body>
</html>
```

---

## Exercício 8.3 – Restrição de papéis com `@PreAuthorize`
**Cenário**
Implemente método `deleteUser` que só pode ser executado por usuários com papel `ADMIN`.

```java
@DeleteMapping("/admin/users/{id}")
@PreAuthorize("hasRole('ADMIN')") // 🟢 Atividade: proteger método
public void deleteUser(@PathVariable Long id) { service.delete(id); }
```

---

## Exercício 8.4 – Configurar CORS para recursos estáticos
**Cenário**
Permita solicitações da origem `https://frontend.example.com` apenas para `/static/**`.

```java
http.cors(cors -> cors.configurationSource(request -> {
    CorsConfiguration cfg = new CorsConfiguration();
    cfg.setAllowedOrigins(List.of("https://frontend.example.com"));
    cfg.setAllowedMethods(List.of("GET"));
    cfg.setAllowedHeaders(List.of("*");
    return cfg;
}));
```

---

## Exercício 8.5 – CSRF protection somente para formulários
**Cenário**
Desative CSRF para APIs REST (`/api/**`) mas mantenha habilitado para páginas HTML.

```java
http
    .csrf(csrf -> csrf
        .ignoringRequestMatchers("/api/**") // 🟢 Atividade: ignorar API
        .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()));
```

---

## Exercício 8.6 – Implementar JWT authentication
**Cenário**
Crie endpoint `/auth/login` que devolve token JWT e filtro que verifica o token nas requisições.

```java
// DTO de login
public record LoginRequest(String username, String password) {}
public record JwtResponse(String token) {}
```

```java
@RestController
@RequestMapping("/auth")
public class AuthController {
    private final AuthenticationManager authManager;
    private final JwtUtil jwtUtil;
    public AuthController(AuthenticationManager authManager, JwtUtil jwtUtil) {
        this.authManager = authManager; this.jwtUtil = jwtUtil;
    }
    @PostMapping("/login")
    public ResponseEntity<JwtResponse> login(@RequestBody LoginRequest req) {
        Authentication auth = authManager.authenticate(new UsernamePasswordAuthenticationToken(req.username(), req.password()));
        String token = jwtUtil.generateToken(auth);
        return ResponseEntity.ok(new JwtResponse(token)); // 🟢 Atividade: devolver token
    }
}
```

```java
public class JwtAuthFilter extends OncePerRequestFilter {
    private final JwtUtil jwtUtil;
    private final UserDetailsService uds;
    public JwtAuthFilter(JwtUtil jwtUtil, UserDetailsService uds) { this.jwtUtil = jwtUtil; this.uds = uds; }
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
        String header = request.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            String username = jwtUtil.extractUsername(token);
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails user = uds.loadUserByUsername(username);
                if (jwtUtil.validateToken(token, user)) {
                    UsernamePasswordAuthenticationToken auth = new UsernamePasswordAuthenticationToken(user, null, user.getAuthorities());
                    auth.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    SecurityContextHolder.getContext().setAuthentication(auth); // 🟡 Atividade: set context
                }
            }
        }
        chain.doFilter(request, response);
    }
}
```

---

## Exercício 8.7 – Restrição de acesso por método HTTP
**Cenário**
Permita `GET` para usuários `USER` e `POST/PUT/DELETE` apenas para `ADMIN` nas rotas `/admin/**`.

```java
http.authorizeHttpRequests(auth -> auth
    .requestMatchers(HttpMethod.GET, "/admin/**").hasRole("USER")
    .requestMatchers(HttpMethod.POST, "/admin/**").hasRole("ADMIN")
    .requestMatchers(HttpMethod.PUT, "/admin/**").hasRole("ADMIN")
    .requestMatchers(HttpMethod.DELETE, "/admin/**").hasRole("ADMIN"));
```

---

## Exercício 8.8 – Interceptor para auditoria de acesso
**Cenário**
Logue usuário autenticado, método HTTP e URL em cada requisição.

```java
@Component
public class AuditInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) {
        String user = SecurityContextHolder.getContext().getAuthentication().getName();
        System.out.println("[AUDIT] " + user + " -> " + req.getMethod() + " " + req.getRequestURI()); // 🟢 Atividade: log
        return true;
    }
}
```

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    private final AuditInterceptor audit;
    public WebConfig(AuditInterceptor audit) { this.audit = audit; }
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(audit).addPathPatterns("/**"); // 🟡 Atividade: registrar interceptor
    }
}
```

---

## Exercício 8.9 – Testar segurança com `MockMvc`
**Cenário**
Escreva teste que verifica acesso negado a `/admin/users/1` para usuário sem papel `ADMIN`.

```java
@AutoConfigureMockMvc
@SpringBootTest
class SecurityMockMvcTest {
    @Autowired private MockMvc mvc;
    @Test
    void userCannotAccessAdmin() throws Exception {
        mvc.perform(get("/admin/users/1").with(httpBasic("user","userpass")))
           .andExpect(status().isForbidden()); // 🟡 Atividade: assert
    }
}
```

---

## Exercício 8.10 – Documentar segurança no OpenAPI
**Cenário**
Adicione esquema `bearerAuth` e marque os controladores com `@SecurityRequirement(name = "bearerAuth")`.

```java
@SecurityScheme(name = "bearerAuth", type = SecurityScheme.Type.HTTP, scheme = "bearer", bearerFormat = "JWT")
@Configuration
public class OpenApiSecurityConfig {
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI().addSecurityItem(new SecurityRequirement().addList("bearerAuth"));
    }
}
```

---

**Como usar**
1. Crie o pacote `com.example.security` e adicione as classes acima.
2. Preencha os *TODO* marcados com 🟢/🟡.
3. Rode a aplicação (`mvn spring-boot:run`) e teste os endpoints com Postman ou curl.

> **Arquivo salvo em** `C:\Users\lucas\OneDrive\Documentos\Cursos\spring-boot-4-spring-7-hibernate-for-beginners\Questoes\Pratica\Capitulo_08_Spring_MVC_Security_Pratico_Questoes.md`
