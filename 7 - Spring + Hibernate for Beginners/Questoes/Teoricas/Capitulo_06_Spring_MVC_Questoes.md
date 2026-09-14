# Questões Teóricas - Capítulo 06: Spring MVC

### Questão 1
O que é o **Thymeleaf** e como ele se integra ao Spring Boot na renderização do lado do servidor (*Server-Side Rendering*)?

<details>
<summary>👀 Ver Resposta</summary>

O Thymeleaf é uma engine de templates Java moderna para a camada de visualização (View). Ele se integra ao Spring MVC parseando páginas HTML estáticas enriquecidas com atributos especiais (`th:*`). O Spring injeta objetos do modelo nesses templates e o Thymeleaf gera HTML puro dinâmico enviado diretamente ao navegador do usuário.
</details>

---

### Questão 2
Explique o fluxo de execução do **`DispatcherServlet`** na arquitetura Spring MVC.

<details>
<summary>👀 Ver Resposta</summary>

O `DispatcherServlet` atua como o controlador frontal (*Front Controller*). Ao receber uma requisição HTTP:
1. Ele consulta o `HandlerMapping` para encontrar o Controller correto.
2. O Controller executa a regra de negócio e popula o objeto `Model`.
3. O Controller retorna o nome lógico de uma View (string).
4. O `DispatcherServlet` envia o nome da View ao `ViewResolver` (Thymeleaf) para localizar o arquivo HTML.
5. A View é renderizada com os dados do `Model` e a resposta HTTP é retornada ao cliente.
</details>

---

### Questão 3
Como funciona o **Form Binding** no Spring MVC com Thymeleaf?

<details>
<summary>👀 Ver Resposta</summary>

O Form Binding associa um objeto Java (POJO) diretamente a um formulário HTML. No Spring Controller, adiciona-se o objeto ao modelo (`model.addAttribute("student", new Student())`). No HTML, o formulário referencia o objeto via `th:object="${student}"` e seus campos são vinculados através do atributo `th:field="*{firstName}"`.
</details>

---

### Questão 4
Para que serve a anotação `@ModelAttribute` nos métodos de um `@Controller` Spring MVC?

<details>
<summary>👀 Ver Resposta</summary>

A anotação `@ModelAttribute` é usada para extrair e rebindar dados de formulários HTML submetidos de volta em um POJO Java passado como parâmetro do método do controller, facilitando a recepção de dados validados.
</details>

---

### Questão 5
Como o **Jakarta Bean Validation** é ativado no Spring MVC e quais são as principais anotações de validação padrão?

<details>
<summary>👀 Ver Resposta</summary>

A validação é ativada adicionando a anotação `@Valid` antes do parâmetro `@ModelAttribute` no método do controller. Suas principais anotações são:
* `@NotNull` / `@NotEmpty` / `@NotBlank`: Garante que o campo não seja nulo/vazio.
* `@Size(min=X, max=Y)`: Valida o tamanho de strings ou coleções.
* `@Min(X)` / `@Max(Y)`: Valida limites numéricos.
* `@Pattern(regexp="...")`: Valida a string contra uma Expressão Regular (Regex).
</details>

---

### Questão 6
Qual é a regra obrigatória referente ao parâmetro `BindingResult` ao realizar validações de formulário no Spring MVC?

<details>
<summary>👀 Ver Resposta</summary>

O objeto `BindingResult` armazena o resultado da validação e os erros encontrados. Ele **deve ser colocado imediatamente após** o parâmetro anotado com `@Valid` na assinatura do método do controller. Se outro parâmetro for colocado entre eles, a aplicação lançará uma exceção de runtime.
</details>

---

### Questão 7
Como remover automaticamente espaços em branco das pontas (*trimming*) de strings enviadas em formulários utilizando o `@InitBinder`?

<details>
<summary>👀 Ver Resposta</summary>

Declara-se um método anotado com `@InitBinder` no controller registrando um editor de propriedades customizado via `WebDataBinder`:
```java
@InitBinder
public void initBinder(WebDataBinder dataBinder) {
    StringTrimmerEditor stringTrimmerEditor = new StringTrimmerEditor(true);
    dataBinder.registerCustomEditor(String.class, stringTrimmerEditor);
}
```
*(O parâmetro `true` converte strings vazias constituídas apenas por espaços para `null`)*.
</details>

---

### Questão 8
Como personalizar mensagens de erro de validação utilizando o arquivo `messages.properties`?

<details>
<summary>👀 Ver Resposta</summary>

Cria-se o arquivo `messages.properties` na pasta `src/main/resources` definindo chaves que seguem a convenção do Spring Validation: `NomeDaAnotacao.nomeDoObjeto.nomeDoCampo` (ex: `typeMismatch.student.freePasses=Deve ser um número válido`).
</details>

---

### Questão 9
Como criar uma **Anotação de Validação Customizada** no Spring MVC?

<details>
<summary>👀 Ver Resposta</summary>

É necessário criar duas estruturas:
1. A anotação Java (ex: `@CourseCode`) anotada com `@Constraint(validatedBy = CourseCodeConstraintValidator.class)`, definindo os métodos `message()`, `value()` e `payload()`.
2. A classe validadora que implementa `ConstraintValidator<CourseCode, String>` contendo a lógica de validação no método `isValid()`.
</details>

---

### Questão 10
O que faz a expressão `${param.error}` ou `${#fields.hasErrors('campo')}` nos templates do Thymeleaf?

<details>
<summary>👀 Ver Resposta</summary>

São expressões que checam a presença de erros durante a renderização. `${#fields.hasErrors('campo')}` verifica se o campo específico falhou na validação e permite exibir a mensagem associada através da tag `th:errors="*{campo}"`.
</details>
