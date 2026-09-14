# Questões Teóricas - Capítulo 04: REST CRUD APIs

### Questão 1
O que é a arquitetura REST (Representational State Transfer) e quais são seus métodos HTTP padrão para operações CRUD?

<details>
<summary>👀 Ver Resposta</summary>

REST é um estilo arquitetural para sistemas distribuídos que utiliza o protocolo HTTP para transferência de dados. Seus métodos padrão para CRUD são:
* **`GET`**: Recupera um ou mais recursos (Leitura).
* **`POST`**: Cria um novo recurso.
* **`PUT`**: Atualiza um recurso existente completamente (substituição).
* **`PATCH`**: Atualiza um recurso existente parcialmente.
* **`DELETE`**: Remove um recurso existente.
</details>

---

### Questão 2
Como funciona a serialização e desserialização de dados em controladores `@RestController` no Spring Boot?

<details>
<summary>👀 Ver Resposta</summary>

O Spring Boot utiliza internamente a biblioteca **Jackson** (ou Jackson 3 no Spring Boot 4) via conversores de mensagem HTTP (`HttpMessageConverter`). Na entrada, o JSON da requisição é desserializado em POJOs Java (via `@RequestBody`). Na saída, os POJOs retornados pelos métodos anotados com `@RestController` são automaticamente serializados em JSON.
</details>

---

### Questão 3
Qual é a diferença entre as anotações `@PathVariable` e `@RequestParam` no mapeamento de requisições REST?

<details>
<summary>👀 Ver Resposta</summary>

* **`@PathVariable`**: Extrai valores diretamente contidos no caminho da URL (URI Path), como em `/api/employees/{employeeId}`.
* **`@RequestParam`**: Extrai parâmetros passados na Query String da URL após o sinal de interrogação, como em `/api/employees?department=IT`.
</details>

---

### Questão 4
O que é o tratamento global de exceções via `@ControllerAdvice` e `@ExceptionHandler` no Spring REST?

<details>
<summary>👀 Ver Resposta</summary>

O `@ControllerAdvice` atua como um interceptador global de exceções para todos os controladores da aplicação. Ao declarar métodos com `@ExceptionHandler(TipoDaExcecao.class)` dentro dessa classe, é possível capturar exceções específicas lançadas em qualquer lugar da aplicação e retornar uma resposta JSON padronizada (ex: `EmployeeErrorResponse`) com status HTTP adequado (ex: 404 Not Found).
</details>

---

### Questão 5
Qual é o papel da interface `JpaRepository` do Spring Data JPA e quais vantagens ela oferece sobre a implementação manual de DAOs com `EntityManager`?

<details>
<summary>👀 Ver Resposta</summary>

A interface `JpaRepository` fornece métodos CRUD completos pré-implementados (como `findAll()`, `findById()`, `save()`, `deleteById()`), eliminando a necessidade de escrever código boilerplate com `EntityManager` e JPQL para operações padrão.
</details>

---

### Questão 6
Como o Spring Data JPA permite a criação de consultas customizadas através de **Query Methods** (Convenção de Nomes)?

<details>
<summary>👀 Ver Resposta</summary>

O Spring Data JPA analisa o nome do método declarado na interface do repositório e gera a consulta JPQL automaticamente. Por exemplo, declarar `List<Employee> findByLastNameOrderByFirstNameAsc(String lastName)` instrui o Spring Data JPA a criar uma query filtrando por sobrenome e ordenando por primeiro nome em ordem ascendente sem escrever nenhuma query manual.
</details>

---

### Questão 7
O que é o **Spring Data REST** (`spring-boot-starter-data-rest`) e qual é a sua proposta?

<details>
<summary>👀 Ver Resposta</summary>

É um módulo do Spring que analisa os repositórios `JpaRepository` da aplicação e expõe automaticamente uma API RESTful CRUD completa para as entidades mapeadas sem a necessidade de escrever nenhum `@RestController` ou classe de Serviço.
</details>

---

### Questão 8
O que é o formato **HATEOAS (Hypermedia As The Engine Of Application State)** e o padrão HAL utilizados pelo Spring Data REST?

<details>
<summary>👀 Ver Resposta</summary>

HATEOAS é um princípio da arquitetura REST onde as respostas da API contêm não apenas os dados do recurso, mas também hiperlinks (URLs) contendo as ações e navegações possíveis a partir daquele ponto. O padrão HAL (*Hypertext Application Language*) é o formato JSON padronizado usado para estruturar esses links (chave `_links`).
</details>

---

### Questão 9
Como customizar a rota base e o nome dos recursos expostos pelo Spring Data REST?

<details>
<summary>👀 Ver Resposta</summary>

A rota base global pode ser definida no `application.properties` via `spring.data.rest.base-path=/api`. O nome específico do recurso e do caminho de uma entidade pode ser alterado na interface do repositório usando a anotação `@RepositoryRestResource(path="members")`.
</details>

---

### Questão 10
Qual é a diferença funcional entre os métodos HTTP `PUT` e `PATCH` ao atualizar um recurso no banco de dados?

<details>
<summary>👀 Ver Resposta</summary>

* **`PUT`**: Substitui o recurso por inteiro. Todos os campos devem ser enviados no JSON; campos omitidos podem ser sobrescritos com valores nulos no banco.
* **`PATCH`**: Realiza uma atualização parcial. Altera apenas os campos explicitamente enviados no payload JSON, mantendo intactos os demais valores já existentes no recurso.
</details>
