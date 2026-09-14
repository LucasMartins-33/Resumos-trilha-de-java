# Questões Práticas – Capítulo 04: REST CRUD APIs

## Exercício 4.1 – Definir endpoint `GET /api/products` que retorna lista de produtos
**Cenário**
Crie um `ProductController` com método que consulta todos os produtos via `ProductService`.

```java
package com.example.demo.controller;

import com.example.demo.model.Product;
import com.example.demo.service.ProductService;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/api/products") // 🟢 Atividade: base path
public class ProductController {
    private final ProductService service;
    public ProductController(ProductService service) { this.service = service; }

    @GetMapping // 🟢 Atividade: listar todos
    public List<Product> getAll() { return service.findAll(); }
}
```

---

## Exercício 4.2 – Implementar endpoint `GET /api/products/{id}`
**Cenário**
Retorne o produto ou 404 se não encontrado.

```java
@GetMapping("/{id}") // 🟢 Atividade: buscar por id
public Product getById(@PathVariable Long id) {
    return service.findById(id).orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
}
```

---

## Exercício 4.3 – Criar endpoint `POST /api/products` para inserir novo produto
**Cenário**
Receba JSON e persista via `ProductService.save`.

```java
@PostMapping // 🟢 Atividade: criar
public Product create(@RequestBody Product product) { return service.save(product); }
```

---

## Exercício 4.4 – Implementar `PUT /api/products/{id}` para atualizar
**Cenário**
Atualize campos e retorne o produto atualizado.

```java
@PutMapping("/{id}") // 🟢 Atividade: atualizar
public Product update(@PathVariable Long id, @RequestBody Product payload) {
    Product existing = service.findById(id).orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
    existing.setName(payload.getName());
    existing.setPrice(payload.getPrice());
    return service.save(existing);
}
```

---

## Exercício 4.5 – Implementar `DELETE /api/products/{id}`
**Cenário**
Remova o produto e retorne `204 No Content`.

```java
@DeleteMapping("/{id}") // 🟢 Atividade: remover
@ResponseStatus(HttpStatus.NO_CONTENT)
public void delete(@PathVariable Long id) { service.deleteById(id); }
```

---

## Exercício 4.6 – Configurar resposta `application/json` e usar `@ResponseBody`
**Cenário**
Garanta que todos os métodos retornem JSON sem necessidade de view.

```java
@RestController // já inclui @ResponseBody
```

---

## Exercício 4.7 – Validar entrada com `@Valid` e Bean Validation
**Cenário**
Adicione constraints ao DTO `ProductDto` (ex.: `@NotBlank name`). Use `@Valid` no controller.

```java
public class ProductDto {
    @NotBlank
    private String name;
    @Positive
    private BigDecimal price;
    // getters/setters
}

@PostMapping
public Product create(@Valid @RequestBody ProductDto dto) { /* map e salvar */ }
```

---

## Exercício 4.8 – Tratar exceções globais com `@ControllerAdvice`
**Cenário**
Crie uma classe que capture `MethodArgumentNotValidException` e devolva erro 400 com detalhes.

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

## Exercício 4.9 – HATEOAS básico com `EntityModel`
**Cenário**
Adicione links self e collection ao recurso `Product`.

```java
@GetMapping("/{id}")
public EntityModel<Product> getById(@PathVariable Long id) {
    Product prod = service.findById(id).orElseThrow(...);
    return EntityModel.of(prod,
        linkTo(methodOn(ProductController.class).getById(id)).withSelfRel(),
        linkTo(methodOn(ProductController.class).getAll()).withRel("products"));
}
```

---

## Exercício 4.10 – Documentar API com Springdoc OpenAPI
**Cenário**
Adicione dependência `springdoc-openapi-starter-webmvc-ui` e exponha `/swagger-ui.html`.

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version>
</dependency>
```

> **Arquivo salvo em** `C:\Users\lucas\OneDrive\Documentos\Cursos\spring-boot-4-spring-7-hibernate-for-beginners\Questoes\Pratica\Capitulo_04_REST_CRUD_Pratico_Questoes.md`
