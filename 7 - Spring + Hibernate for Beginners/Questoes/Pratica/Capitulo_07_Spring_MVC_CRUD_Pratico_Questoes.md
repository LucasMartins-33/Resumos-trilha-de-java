# Questões Práticas – Capítulo 07: Spring MVC CRUD

## Exercício 7.1 – Criar endpoint `GET /clientes` que lista clientes
**Cenário**
Implemente um `ClienteController` que devolve a lista de clientes usando `ClienteService.findAll()`.

```java
package com.example.demo.controller;

import com.example.demo.model.Cliente;
import com.example.demo.service.ClienteService;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/clientes") // 🟢 Atividade: base path
public class ClienteController {
    private final ClienteService service;
    public ClienteController(ClienteService service) { this.service = service; }

    @GetMapping // 🟢 Atividade: listar todos
    public List<Cliente> listar() { return service.findAll(); }
}
```

---

## Exercício 7.2 – Endpoint `GET /clientes/{id}`
**Cenário**
Retorne o cliente ou lance `ResponseStatusException(HttpStatus.NOT_FOUND)` caso não exista.

```java
@GetMapping("/{id}") // 🟢 Atividade: buscar por id
public Cliente buscar(@PathVariable Long id) {
    return service.findById(id).orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
}
```

---

## Exercício 7.3 – Criar endpoint `POST /clientes` para cadastro
**Cenário**
Receba um JSON `ClienteDto`, converta para entidade e persista via `ClienteService.save`.

```java
@PostMapping // 🟢 Atividade: criar novo cliente
public Cliente criar(@RequestBody ClienteDto dto) {
    Cliente c = new Cliente();
    c.setNome(dto.getNome());
    c.setEmail(dto.getEmail());
    return service.save(c);
}
```

---

## Exercício 7.4 – Atualizar cliente com `PUT /clientes/{id}`
**Cenário**
Carregue o cliente existente, copie os campos do DTO e salve.

```java
@PutMapping("/{id}") // 🟢 Atividade: atualizar
public Cliente atualizar(@PathVariable Long id, @RequestBody ClienteDto dto) {
    Cliente existente = service.findById(id).orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
    existente.setNome(dto.getNome());
    existente.setEmail(dto.getEmail());
    return service.save(existente);
}
```

---

## Exercício 7.5 – Remover cliente com `DELETE /clientes/{id}`
**Cenário**
Exclua o cliente e retorne `204 No Content`.

```java
@DeleteMapping("/{id}") // 🟢 Atividade: remover
@ResponseStatus(HttpStatus.NO_CONTENT)
public void remover(@PathVariable Long id) { service.deleteById(id); }
```

---

## Exercício 7.6 – Validação de entrada com Bean Validation
**Cenário**
Adicione constraints (`@NotBlank`, `@Email`) ao `ClienteDto` e use `@Valid` no controller.

```java
public class ClienteDto {
    @NotBlank
    private String nome;
    @Email
    private String email;
    // getters/setters
}

@PostMapping
public Cliente criar(@Valid @RequestBody ClienteDto dto) { /* ... */ }
```

---

## Exercício 7.7 – Tratamento global de exceções com `@ControllerAdvice`
**Cenário**
Capture `MethodArgumentNotValidException` e retorne JSON com detalhes de erro.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String,String> handleValidation(MethodArgumentNotValidException ex) {
        return ex.getBindingResult().getFieldErrors().stream()
            .collect(Collectors.toMap(FieldError::getField, FieldError::getDefaultMessage));
    }
}
```

---

## Exercício 7.8 – HATEOAS básico com `EntityModel`
**Cenário**
Adicione links `self` e `clientes` ao recurso retornado.

```java
@GetMapping("/{id}")
public EntityModel<Cliente> buscar(@PathVariable Long id) {
    Cliente c = service.findById(id).orElseThrow(...);
    return EntityModel.of(c,
        linkTo(methodOn(ClienteController.class).buscar(id)).withSelfRel(),
        linkTo(methodOn(ClienteController.class).listar()).withRel("clientes"));
}
```

---

## Exercício 7.9 – Documentar API com SpringDoc OpenAPI
**Cenário**
Adicione dependência `springdoc-openapi-starter-webmvc-ui` e exponha `/swagger-ui.html`.

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

---

## Exercício 7.10 – Teste de integração com `@SpringBootTest` e `TestRestTemplate`
**Cenário**
Verifique que `GET /clientes` retorna lista vazia inicialmente e que `POST` adiciona um cliente.

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ClienteIntegrationTest {
    @Autowired private TestRestTemplate rest;

    @Test
    void crudFlow() {
        // Listar vazia
        ResponseEntity<List> resp = rest.getForEntity("/clientes", List.class);
        Assertions.assertTrue(resp.getBody().isEmpty());
        // Criar cliente
        ClienteDto dto = new ClienteDto();
        dto.setNome("Ana"); dto.setEmail("ana@example.com");
        Cliente created = rest.postForObject("/clientes", dto, Cliente.class);
        Assertions.assertNotNull(created.getId());
    }
}
```

---

**Como usar**
1. Crie o pacote `com.example.demo` com sub‑pacotes `controller`, `service`, `model` e `dto`.
2. Preencha os blocos marcados com 🟢/🟡.
3. Rode `mvn spring-boot:run` e teste os endpoints com Postman ou curl.

> **Arquivo salvo em** `C:\Users\lucas\OneDrive\Documentos\Cursos\spring-boot-4-spring-7-hibernate-for-beginners\Questoes\Pratica\Capitulo_07_Spring_MVC_CRUD_Pratico_Questoes.md`
