# Questões Teóricas - Capítulo 09: JPA / Hibernate Advanced Mappings

### Questão 1
O que são as definições de **Owning Side (Lado Proprietário)** e **Inverse Side (Lado Inverso)** nos mapeamentos de relacionamentos JPA?

<details>
<summary>👀 Ver Resposta</summary>

* **Owning Side**: É a entidade que possui a coluna física da Chave Estrangeira (FK) na tabela do banco de dados e mapeia a relação usando `@JoinColumn`. É responsável por atualizar o banco de dados.
* **Inverse Side**: É a entidade do outro lado do relacionamento que aponta de volta para o Owning Side através do atributo `mappedBy`.
</details>

---

### Questão 2
Quais são os quatro estados principais do ciclo de vida de uma entidade gerenciada pelo `EntityManager` no JPA?

<details>
<summary>👀 Ver Resposta</summary>

1. **Transient / New**: Objeto instanciado via `new` que ainda não possui ID no banco nem é gerenciado pelo `EntityManager`.
2. **Persistent / Managed**: Objeto associado ao contexto de persistência. Alterações nos atributos são sincronizadas no banco automaticamente.
3. **Detached**: Objeto com ID existente no banco, mas cuja sessão do `EntityManager` foi encerrada.
4. **Removed**: Objeto agendado para exclusão no banco de dados.
</details>

---

### Questão 3
Explique a diferença entre os tipos de cascata `CascadeType.PERSIST` e `CascadeType.REMOVE`.

<details>
<summary>👀 Ver Resposta</summary>

* **`CascadeType.PERSIST`**: Ao salvar a entidade pai, o JPA salva automaticamente as entidades filhas vinculadas em memória.
* **`CascadeType.REMOVE`**: Ao deletar a entidade pai, o JPA exclui automaticamente todas as entidades filhas vinculadas no banco de dados.
</details>

---

### Questão 4
Por que o uso de `CascadeType.REMOVE` ou `CascadeType.ALL` deve ser evitado em relacionamentos N:N (`@ManyToMany`)?

<details>
<summary>👀 Ver Resposta</summary>

Porque em um relacionamento `@ManyToMany` (ex: Cursos e Alunos), as entidades possuem ciclo de vida independente. Apagar um Aluno com cascata `REMOVE` iria deletar todos os Cursos associados a ele no banco de dados, excluindo dados de outros alunos matriculados nesses mesmos cursos.
</details>

---

### Questão 5
Diferencie as estratégias de carregamento `FetchType.EAGER` e `FetchType.LAZY` no JPA e quais os seus valores padrão.

<details>
<summary>👀 Ver Resposta</summary>

* **`EAGER`**: Carrega a entidade principal e todos os seus relacionamentos associados imediatamente em uma única consulta. (Padrão em `@OneToOne` e `@ManyToOne`).
* **`LAZY`**: Carrega apenas a entidade principal. Relacionamentos associados só são buscados no banco quando seus respectivos getters são invocados. (Padrão em `@OneToMany` e `@ManyToMany`).
</details>

---

### Questão 6
O que causa a exceção **`LazyInitializationException`** no Hibernate e qual a forma ideal de resolvê-la mantendo o relacionamento como `LAZY`?

<details>
<summary>👀 Ver Resposta</summary>

A exceção ocorre quando o código tenta acessar um atributo/coleção com carregamento `LAZY` após a sessão do Hibernate ou `EntityManager` ter sido fechada. A melhor forma de resolver é usar uma consulta JPQL customizada com **`JOIN FETCH`**, que carrega os dados relacionados em uma única query otimizada sem alterar o mapeamento global para `EAGER`.
</details>

---

### Questão 7
Como funciona a cláusula **`JOIN FETCH`** no JPQL?

<details>
<summary>👀 Ver Resposta</summary>

O `JOIN FETCH` instrui o Hibernate a realizar um INNER JOIN na tabela relacionada no banco de dados e inicializar completamente os objetos filhos no momento da execução da consulta principal, prevenindo a `LazyInitializationException` mesmo quando o relacionamento está configurado como `FetchType.LAZY`.
</details>

---

### Questão 8
Para que serve o atributo `mappedBy` na anotação `@OneToOne` ou `@OneToMany`?

<details>
<summary>👀 Ver Resposta</summary>

O `mappedBy` indica que o relacionamento é **bidirecional** e especifica o nome do atributo Java na classe proprietária (*Owning Side*) que gerencia a chave estrangeira. Ele diz ao Hibernate para não criar uma nova coluna ou tabela de junção, apenas reutilizar o mapeamento do outro lado.
</details>

---

### Questão 9
Como é estruturado um mapeamento N:N (`@ManyToMany`) em tabelas de banco de dados e como mapeá-lo com a anotação `@JoinTable` no JPA?

<details>
<summary>👀 Ver Resposta</summary>

Banco de dados exige uma **Tabela de Junção (Join Table)** com duas Chaves Estrangeiras apontando para as tabelas principais. No JPA, mapeia-se com `@JoinTable`:
```java
@JoinTable(
    name = "course_student",
    joinColumns = @JoinColumn(name = "course_id"),
    inverseJoinColumns = @JoinColumn(name = "student_id")
)
```
</details>

---

### Questão 10
Como excluir uma entidade com relacionamento bidirecional `@OneToOne` preservando a entidade associada (ex: apagar `InstructorDetail` mas manter o `Instructor`)?

<details>
<summary>👀 Ver Resposta</summary>

É necessário:
1. Remover o `CascadeType.REMOVE` ou `ALL` da entidade.
2. Desfazer o vínculo em memória antes de excluir chamando `tempInstructorDetail.getInstructor().setInstructorDetail(null)`.
3. Chamar `entityManager.remove(tempInstructorDetail)`.
</details>
