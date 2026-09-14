# Questões Práticas – Capítulo 06: Spring MVC

## Exercício 6.1 – Configurar `ViewResolver` para Thymeleaf
**Cenário**
Crie a configuração que habilita Thymeleaf como view engine e define o prefix `/templates/` e suffix `.html`.

```java
package com.example.mvc.config;

import org.springframework.context.annotation.*;
import org.thymeleaf.spring5.SpringTemplateEngine;
import org.thymeleaf.spring5.view.ThymeleafViewResolver;
import org.thymeleaf.templateresolver.SpringResourceTemplateResolver;

@Configuration
public class ThymeleafConfig {

    @Bean // 🟢 Atividade: resolver de templates
    public SpringResourceTemplateResolver templateResolver() {
        SpringResourceTemplateResolver resolver = new SpringResourceTemplateResolver();
        resolver.setPrefix("classpath:/templates/");
        resolver.setSuffix(".html");
        resolver.setTemplateMode("HTML");
        resolver.setCharacterEncoding("UTF-8");
        return resolver;
    }

    @Bean // 🟢 Atividade: engine
    public SpringTemplateEngine templateEngine(SpringResourceTemplateResolver resolver) {
        SpringTemplateEngine engine = new SpringTemplateEngine();
        engine.setTemplateResolver(resolver);
        return engine;
    }

    @Bean // 🟢 Atividade: view resolver
    public ThymeleafViewResolver viewResolver(SpringTemplateEngine engine) {
        ThymeleafViewResolver vr = new ThymeleafViewResolver();
        vr.setTemplateEngine(engine);
        vr.setCharacterEncoding("UTF-8");
        return vr;
    }
}
```

---

## Exercício 6.2 – Controlador que devolve `ModelAndView`
**Cenário**
Implemente `HomeController` que retorna a view `home.html` com atributo `message`.

```java
package com.example.mvc.controller;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class HomeController {
    @GetMapping("/") // 🟢 Atividade: mapeamento raiz
    public String home(Model model) {
        model.addAttribute("message", "Bem‑vindo ao Spring MVC!");
        return "home"; // resolve para home.html
    }
}
```

---

## Exercício 6.3 – Formulário HTML com `@ModelAttribute`
**Cenário**
Crie página `form.html` que envia dados para `POST /users` e preenche um bean `UserForm`.

```java
package com.example.mvc.model;

public class UserForm {
    private String username;
    private String email;
    // getters/setters // 🟢 Atividade: POJO simples
}
```

```java
@PostMapping("/users")
public String submit(@ModelAttribute UserForm form, Model model) {
    model.addAttribute("user", form);
    return "result"; // exibe result.html
}
```

---

## Exercício 6.4 – Validar formulário com Bean Validation
**Cenário**
Adicione anotações `@NotBlank` e `@Email` ao `UserForm` e use `@Valid` no controlador.

```java
public class UserForm {
    @NotBlank
    private String username;
    @Email
    private String email;
    // getters/setters
}

@PostMapping("/users")
public String submit(@Valid @ModelAttribute UserForm form, BindingResult br, Model model) {
    if (br.hasErrors()) {
        return "form"; // volta ao formulário
    }
    model.addAttribute("user", form);
    return "result";
}
```

---

## Exercício 6.5 – Redirecionamento Post‑Redirect‑Get
**Cenário**
Depois de salvar o usuário, redirecione para `/users/{username}`.

```java
@PostMapping("/users")
public String submit(@Valid @ModelAttribute UserForm form, BindingResult br) {
    if (br.hasErrors()) return "form";
    // salvar usuário (omissão)
    return "redirect:/users/" + form.getUsername(); // 🟢 Atividade: PRG
}
```

---

## Exercício 6.6 – Controlador REST ao lado do MVC
**Cenário**
Adicione um controlador que expõe `/api/time` retornando `LocalDateTime` em JSON.

```java
@RestController
@RequestMapping("/api")
public class TimeRestController {
    @GetMapping("/time")
    public LocalDateTime now() { return LocalDateTime.now(); } // 🟢 Atividade: endpoint JSON
}
```

---

## Exercício 6.7 – Interceptar requisições com `HandlerInterceptor`
**Cenário**
Logue URL e tempo de início antes da execução da controller.

```java
@Component
public class LoggingInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse res, Object handler) {
        req.setAttribute("startTime", System.currentTimeMillis());
        System.out.println("[LOG] Incoming " + req.getMethod() + " " + req.getRequestURI()); // 🟢 Atividade: log
        return true;
    }
    @Override
    public void afterCompletion(HttpServletRequest req, HttpServletResponse res, Object handler, Exception ex) {
        long start = (Long) req.getAttribute("startTime");
        System.out.println("[LOG] Completed in " + (System.currentTimeMillis() - start) + " ms"); // 🟢 Atividade: log
    }
}
```

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    private final LoggingInterceptor interceptor;
    public WebConfig(LoggingInterceptor interceptor) { this.interceptor = interceptor; }
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(interceptor).addPathPatterns("/**"); // 🟡 Atividade: registrar
    }
}
```

---

## Exercício 6.8 – Configurar `MultipartResolver` para upload de arquivos
**Cenário**
Permita upload de imagem via `POST /upload` e salve em `/tmp/uploads`.

```java
@Bean // 🟢 Atividade: resolver multipart
public CommonsMultipartResolver multipartResolver() {
    CommonsMultipartResolver resolver = new CommonsMultipartResolver();
    resolver.setDefaultEncoding("utf-8");
    resolver.setMaxUploadSize(5 * 1024 * 1024);
    return resolver;
}
```

```java
@PostMapping("/upload")
public String handle(@RequestParam("file") MultipartFile file) throws IOException {
    File dest = new File("/tmp/uploads/" + file.getOriginalFilename());
    file.transferTo(dest);
    return "uploadSuccess"; // 🟢 Atividade: view de sucesso
}
```

---

## Exercício 6.9 – Configurar `LocaleResolver` e internacionalização
**Cenário**
Suporte a português (pt_BR) e inglês (en_US) com parâmetros `?lang=pt_BR`.

```java
@Bean // 🟢 Atividade: resolver locale
public LocaleResolver localeResolver() {
    SessionLocaleResolver slr = new SessionLocaleResolver();
    slr.setDefaultLocale(Locale.US);
    return slr;
}

@Bean // 🟢 Atividade: interceptor de mudança de locale
public LocaleChangeInterceptor localeChangeInterceptor() {
    LocaleChangeInterceptor lci = new LocaleChangeInterceptor();
    lci.setParamName("lang");
    return lci;
}
```

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(localeChangeInterceptor());
}
```

---

## Exercício 6.10 – Testar controlador MVC com `MockMvc`
**Cenário**
Escreva teste que verifica o retorno da view `home` e o atributo `message`.

```java
@AutoConfigureMockMvc
@SpringBootTest
class HomeControllerTest {
    @Autowired private MockMvc mvc;
    @Test
    void homePage() throws Exception {
        mvc.perform(get("/"))
           .andExpect(status().isOk())
           .andExpect(view().name("home")) // 🟢 Atividade: checar view
           .andExpect(model().attributeExists("message"));
    }
}
```

---

**Como usar**
1. Crie o pacote `com.example.mvc` com sub‑pacotes `config`, `controller`, `model`.
2. Preencha os blocos marcados com 🟢/🟡.
3. Rode a aplicação `mvn spring-boot:run` e acesse `http://localhost:8080/`.

> **Arquivo salvo em** `C:\Users\lucas\OneDrive\Documentos\Cursos\spring-boot-4-spring-7-hibernate-for-beginners\Questoes\Pratica\Capitulo_06_Spring_MVC_Pratico_Questoes.md`
