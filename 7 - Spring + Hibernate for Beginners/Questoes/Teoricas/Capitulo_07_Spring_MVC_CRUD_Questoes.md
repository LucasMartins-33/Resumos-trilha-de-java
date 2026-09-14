# Questões Teóricas - Capítulo 07: Spring MVC CRUD

### Questão 1
O que é o padrão de projeto **PRG (Post/Redirect/Get)** e por que ele é indispensável em aplicações web Spring MVC?

> [!faq]- Resposta
> O PRG é um padrão de desenvolvimento web onde, após processar uma requisição de gravação (`POST`), o servidor responde com um redirecionamento HTTP (`302 Redirect`) instruindo o navegador a fazer uma requisição `GET` para uma nova página. Ele previne a submissão duplicada de dados caso o usuário recarregue a página (pressionando F5).

---

### Questão 2
Como o Spring MVC diferencia uma operação de **Criação (INSERT)** de uma operação de **Atualização (UPDATE)** ao chamar o método `save()` do `JpaRepository`?

> [!faq]- Resposta
> O Spring MVC utiliza um campo de entrada oculto no formulário Thymeleaf (`<input type="hidden" th:field="*{id}" />`).
> * Se o `id` for nulo ou `0`, o Spring Data JPA entende que é um novo registro e executa um `INSERT`.
> * Se o `id` já contiver um valor existente, o JPA carrega e executa um `UPDATE`.

---

### Questão 3
Como realizar a ordenação automática de registros retornados de um banco de dados utilizando a convenção de nomes de repositórios do Spring Data JPA?

> [!faq]- Resposta
> Basta declarar na interface do repositório um método seguindo a sintaxe do Spring Data JPA, como `public List<Employee> findAllByOrderByLastNameAsc()`. O Spring gerará a consulta SQL com a cláusula `ORDER BY last_name ASC` automaticamente.

---

### Questão 4
Como criar um link de ação no Thymeleaf passando um parâmetro de ID da entidade na URL para edição de um formulário?

> [!faq]- Resposta
> Utiliza-se a sintaxe de construtor de URLs do Thymeleaf `@`:
> ```html
> <a th:href="@{/employees/showFormForUpdate(employeeId=${tempEmployee.id})}" 
>    class="btn btn-info btn-sm">Update</a>
> ```

---

### Questão 5
Como implementar uma confirmação de exclusão em JavaScript diretamente no botão de deleção em um template Thymeleaf?

> [!faq]- Resposta
> Utiliza-se o evento `onclick` integrado com a função nativa do JavaScript `confirm()`:
> ```html
> <a th:href="@{/employees/delete(employeeId=${tempEmployee.id})}" 
>    class="btn btn-danger btn-sm"
>    onclick="if (!(confirm('Are you sure you want to delete this employee?'))) return false;">Delete</a>
> ```

---

### Questão 6
Qual é o fluxo completo de camadas em uma arquitetura Spring MVC CRUD desde a requisição do navegador até o banco de dados?

> [!faq]- Resposta
> 1. Browser faz requisição HTTP.
> 2. `EmployeeController` recebe a requisição.
> 3. Controller delega o processamento ao `EmployeeService`.
> 4. Service aciona o `EmployeeRepository` (Spring Data JPA).
> 5. Repositório consulta/persiste no Banco de Dados MySQL.
> 6. Dados retornam da camada Service para o Controller.
> 7. Controller popula o `Model` e direciona para a View Thymeleaf (`list-employees.html`).

---

### Questão 7
Por que a camada de Serviço (`Service Layer`) deve existir entre a camada de Controlador (`Controller`) e a camada de Repositório (`Repository`)?

> [!faq]- Resposta
> A camada de Serviço atua como uma fachada (*Facade*) para regras de negócio e limites transacionais. Ela permite integrar múltiplos repositórios em uma mesma transação, implementar regras de validação complexas e isolar o controlador da infraestrutura direta de persistência.

---

### Questão 8
Como redirecionar o usuário para uma rota específica no Spring Controller após processar o salvamento de um formulário?

> [!faq]- Resposta
> Retorna-se a string com o prefixo `redirect:` no método do controlador:
> ```java
> @PostMapping("/save")
> public String saveEmployee(@ModelAttribute("employee") Employee theEmployee) {
>     employeeService.save(theEmployee);
>     return "redirect:/employees/list";
> }
> ```

---

### Questão 9
Como incluir recursos estáticos (CSS, JavaScript, imagens) como o Bootstrap 5 em uma aplicação Spring Boot com Thymeleaf?

> [!faq]- Resposta
> Os arquivos estáticos são colocados na pasta `src/main/resources/static`. No Thymeleaf, eles são referenciados usando a sintaxe `@`:
> `<link rel="stylesheet" th:href="@{/css/demo.css}" />` ou apontando para um CDN externo.

---

### Questão 10
O que acontece se o campo oculta de `id` (`<input type="hidden" th:field="*{id}" />`) for esquecido na tela de edição de um registro?

> [!faq]- Resposta
> Ao submeter o formulário de atualização, o objeto `Employee` chegará ao controller com o `id` zerado/nulo. Como consequência, o `save()` do Spring Data JPA interpretará a edição como uma tentativa de inclusão e criará um **novo registro duplicado** no banco em vez de atualizar o registro existente.
