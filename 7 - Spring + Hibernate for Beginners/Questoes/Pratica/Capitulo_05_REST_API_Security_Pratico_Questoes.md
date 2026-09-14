# Questões Práticas – Capítulo 05: Segurança de APIs REST

## Exercício 5.1 – Configurar autenticação HTTP Basic
**Cenário**
Adicione segurança básica usando `username=user` e `password=secret` para todos os endpoints `/api/**`.

```java
package com.example.security;

import org.springframework.context.annotation.*;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.core.userdetails.*;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

@Configuration
public class BasicSecurityConfig {

    @Bean // 🟢 Atividade: gerenciar usuários em memória
    public UserDetailsService users() {
        UserDetails user = User.withDefaultPasswordEncoder()
            .username("user")
            .password("secret")
            .roles("USER")
            .build();
        return new InMemoryUserDetailsManager(user);
    }

    @Bean // 🟢 Atividade: definir filtro de segurança
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/**").authenticated()
                .anyRequest().permitAll())
            .httpBasic(); // 🟢 Atividade: habilitar HTTP Basic
        return http.build();
    }
}
```

---

## Exercício 5.2 – Restrição de papéis com `@PreAuthorize`
**Cenário**
Crie dois papéis `ADMIN` e `USER`. O endpoint `DELETE /api/products/{id}` só pode ser acessado por `ADMIN`.

```java
package com.example.security;

import org.springframework.context.annotation.*;
import org.springframework.security.config.annotation.method.configuration.EnableMethodSecurity;
import org.springframework.security.core.userdetails.*;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;

@Configuration
@EnableMethodSecurity // 🟢 Atividade: habilitar @PreAuthorize
public class MethodSecurityConfig {
    @Bean
    public UserDetailsService users() {
        UserDetails admin = User.withDefaultPasswordEncoder()
            .username("admin")
            .password("adminpass")
            .roles("ADMIN")
            .build();
        UserDetails user = User.withDefaultPasswordEncoder()
            .username("user")
            .password("userpass")
            .roles("USER")
            .build();
        return new InMemoryUserDetailsManager(admin, user);
    }
}
```

```java
// No controller
@DeleteMapping("/api/products/{id}")
@PreAuthorize("hasRole('ADMIN')") // 🟡 Atividade: proteger método
public void delete(@PathVariable Long id) { service.deleteById(id); }
```

---

## Exercício 5.3 – Autenticação JWT com Spring Security
**Cenário**
Implemente login `/api/auth/login` que devolve um token JWT. As demais rotas verificam o token.

```java
// 1️⃣ DTO de login
public record LoginRequest(String username, String password) {}
public record JwtResponse(String token) {}
```

```java
// 2️⃣ Controller de autenticação
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    private final AuthenticationManager authManager;
    private final JwtUtil jwtUtil;
    public AuthController(AuthenticationManager authManager, JwtUtil jwtUtil) {
        this.authManager = authManager;
        this.jwtUtil = jwtUtil;
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
// 3️⃣ Filter que valida o token em cada requisição
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    private final JwtUtil jwtUtil;
    private final UserDetailsService userDetailsService;
    public JwtAuthenticationFilter(JwtUtil jwtUtil, UserDetailsService uds) { this.jwtUtil = jwtUtil; this.userDetailsService = uds; }
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
        String header = request.getHeader("Authorization");
        if (header != null && header.startsWith("Bearer ")) {
            String token = header.substring(7);
            String username = jwtUtil.extractUsername(token);
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                if (jwtUtil.validateToken(token, userDetails)) {
                    UsernamePasswordAuthenticationToken auth = new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                    auth.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    SecurityContextHolder.getContext().setAuthentication(auth); // 🟡 Atividade: set security context
                }
            }
        }
        chain.doFilter(request, response);
    }
}
```

---

## Exercício 5.4 – Proteger rotas com `SecurityMatcher` baseado em HTTP method
**Cenário**
Permita somente `GET` para usuários comuns e `POST/PUT/DELETE` apenas para `ADMIN`.

```java
http.authorizeHttpRequests(auth -> auth
    .requestMatchers(HttpMethod.GET, "/api/**").hasRole("USER")
    .requestMatchers(HttpMethod.POST, "/api/**").hasRole("ADMIN")
    .requestMatchers(HttpMethod.PUT, "/api/**").hasRole("ADMIN")
    .requestMatchers(HttpMethod.DELETE, "/api/**").hasRole("ADMIN")
);
```

---

## Exercício 5.5 – Implementar Refresh Token
**Cenário**
Crie endpoint `/api/auth/refresh` que recebe um refresh token válido e devolve um novo access token.

```java
@PostMapping("/refresh")
public ResponseEntity<JwtResponse> refresh(@RequestBody RefreshRequest req) {
    // validar refresh token e gerar novo JWT
    String newToken = jwtUtil.generateAccessToken(req.refreshToken());
    return ResponseEntity.ok(new JwtResponse(newToken)); // 🟢 Atividade: resposta
}
```

---

## Exercício 5.6 – Configurar CORS para API pública
**Cenário**
Permita chamadas da origem `https://myfrontend.com` apenas para `/api/public/**`.

```java
http.cors(cors -> cors.configurationSource(request -> {
    CorsConfiguration cfg = new CorsConfiguration();
    cfg.setAllowedOrigins(List.of("https://myfrontend.com"));
    cfg.setAllowedMethods(List.of("GET","POST"));
    cfg.setAllowedHeaders(List.of("*"));
    return cfg;
}));
```

---

## Exercício 5.7 – Habilitar CSRF somente para métodos não‑GET
**Cenário**
Desative CSRF para API JSON, mas mantenha para formulários web.

```java
http.csrf(csrf -> csrf
    .ignoringRequestMatchers("/api/**") // 🟢 Atividade: ignorar API
    .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse()));
```

---

## Exercício 5.8 – Testar segurança com `MockMvc`
**Cenário**
Escreva teste que verifica acesso negado a `DELETE /api/products/1` para usuário `USER`.

```java
@AutoConfigureMockMvc
@SpringBootTest
class SecurityMockMvcTest {
    @Autowired private MockMvc mvc;
    @Test
    void userCannotDelete() throws Exception {
        mvc.perform(delete("/api/products/1")
                .with(httpBasic("user","userpass")))
            .andExpect(status().isForbidden()); // 🟡 Atividade: assert
    }
}
```

---

## Exercício 5.9 – Auditar requisições usando `HandlerInterceptor`
**Cenário**
Logue método, URI e usuário autenticado para cada request.

```java
@Component
public class AuditInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        String user = SecurityContextHolder.getContext().getAuthentication().getName();
        System.out.println("[AUDIT] " + user + " -> " + request.getMethod() + " " + request.getRequestURI()); // 🟢 Atividade: log
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
        registry.addInterceptor(audit).addPathPatterns("/api/**"); // 🟡 Atividade: registrar interceptor
    }
}
```

---

## Exercício 5.10 – Documentar segurança com OpenAPI `springdoc`
**Cenário**
Adicione anotações `@SecurityRequirement(name = "bearerAuth")` nos controladores e configure esquema no `OpenAPI` bean.

```java
@SecurityScheme(name = "bearerAuth", type = SecurityScheme.Type.HTTP, scheme = "bearer", bearerFormat = "JWT")
@Configuration
public class OpenApiConfig {
    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI().addSecurityItem(new SecurityRequirement().addList("bearerAuth"));
    }
}
```

---

**Como usar**
1. Crie o pacote `com.example.security` e adicione as classes acima.
2. Preencha os TODO marcados com 🟢 ou 🟡.
3. Teste a aplicação com `mvn spring-boot:run` e use Postman/Insomnia para validar tokens.

> **Arquivo salvo em** `C:\Users\lucas\OneDrive\Documentos\Cursos\spring-boot-4-spring-7-hibernate-for-beginners\Questoes\Pratica\Capitulo_05_REST_API_Security_Pratico_Questoes.md`
