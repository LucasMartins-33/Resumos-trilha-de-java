# Capítulo 09: JPA / Hibernate Advanced Mappings

Este resumo cobre os relacionamentos avançados entre entidades no **JPA / Hibernate** utilizando **Spring Boot 4**:
* `@OneToOne` (Unidirecional e Bidirecional)
* `@OneToMany` / `@ManyToOne` (Bidirecional e Unidirecional)
* `@ManyToMany` (Bidirecional com Tabela de Junção/Join Table)
* Conceitos fundamentais de Banco de Dados (Chaves Primárias, Chaves Estrangeiras/Foreign Keys, Integridade Referencial)
* Ciclo de Vida das Entidades JPA (Transient, Persistent/Managed, Detached, Removed)
* Tipos de Cascata (`CascadeType.ALL`, `PERSIST`, `MERGE`, `REMOVE`, `REFRESH`, `DETACH`)
* Estratégias de Carregamento (`FetchType.EAGER` vs `FetchType.LAZY`) e resolução de `LazyInitializationException` via **JOIN FETCH (JPQL)**.

---

## 📌 1. Conceitos Fundamentais de Banco de Dados e Mapeamento JPA

Nos bancos de dados relacionais, as tabelas se conectam através de **Foreign Keys (FK)** que referenciam as **Primary Keys (PK)** de outras tabelas. No JPA, mapeamos essas conexões relacionais para POJOs orientados a objetos.

### Termos-Chave:
* **Owning Side (Lado Proprietário)**: É o lado do relacionamento que possui a coluna física da chave estrangeira (`@JoinColumn`) na tabela do banco de dados.
* **Inverse Side (Lado Inverso/Referenciado)**: É o lado que aponta de volta para o *Owning Side* usando o atributo `mappedBy` (não possui a chave estrangeira no banco).
* **Integridade Referencial**: Restrição que impede a existência de chaves estrangeiras apontando para registros primários inexistentes ou a exclusão acidental de registros pais vinculados a filhos.

---

## 📌 2. Ciclo de Vida da Entidade JPA (Entity Lifecycle)

O Hibernate/JPA gerencia entidades através de um conjunto de estados no `EntityManager`:

```
 [ New / Transient ] -- (persist / save) --> [ Persistent / Managed ]
          ^                                              |
          | (merge)                                      | (remove)
          |                                              v
  [ Detached ] <---- (close / clear / detach) ---- [ Removed ]
```

1. **Transient / New**: Instanciado com `new`, ainda não associado ao `EntityManager` ou à tabela.
2. **Persistent / Managed**: Associado ao contexto de persistência do `EntityManager`. Qualquer alteração via setters é sincronizada no banco automaticamente no `flush`/`commit`.
3. **Detached**: Já possui ID no banco, mas a sessão/`EntityManager` foi encerrada. Alterações não são mais rastreadas automaticamente até chamar o `merge()`.
4. **Removed**: Marcado para exclusão. Será deletado do banco no próximo `flush`/`commit`.

---

## 📌 3. Cascade Types (Efeito em Cascata)

Define como operações executadas numa entidade pai devem se propagar para as entidades associadas.

| CascadeType | Descrição |
| :--- | :--- |
| **`CascadeType.PERSIST`** | Ao salvar/persistir a entidade pai, salva automaticamente os filhos associados. |
| **`CascadeType.REMOVE`** | Ao deletar a entidade pai, deleta automaticamente todos os filhos associados. |
| **`CascadeType.MERGE`** | Ao atualizar/reanexar a entidade pai no banco, atualiza os filhos. |
| **`CascadeType.REFRESH`** | Recarrega dados do banco para o pai e para os filhos associados. |
| **`CascadeType.DETACH`** | Se o pai for desanexado da sessão, desanexa também os filhos. |
| **`CascadeType.ALL`** | Aplica **todas** as operações acima em cascata. |

> [!WARNING]
> **Cuidado com Cascading Delete (`REMOVE` / `ALL`)**:
> Nunca aplique `CascadeType.REMOVE` ou `CascadeType.ALL` em relacionamentos `@ManyToMany` ou em casos onde o registro filho tem ciclo de vida independente (ex: deletar um Aluno **NÃO** deve deletar o Curso, e deletar um Curso **NÃO** deve deletar o Aluno).

---

## 📌 4. Fetch Types: EAGER vs LAZY e a `LazyInitializationException`

* **`FetchType.EAGER`**: Carrega a entidade principal e todas as entidades associadas imediatamente em uma única consulta ou em subconsultas automáticas.
* **`FetchType.LAZY`**: Carrega apenas a entidade principal. Os relacionamentos dependentes são carregados do banco **sob demanda** (apenas no momento em que um getter como `getCourses()` é chamado).

### Padrões de FetchType do JPA:
| Mapeamento | FetchType Padrão |
| :--- | :--- |
| `@OneToOne` | `EAGER` |
| `@ManyToOne` | `EAGER` |
| `@OneToMany` | `LAZY` |
| `@ManyToMany` | `LAZY` |

> [!IMPORTANT]
> **A Exceção `LazyInitializationException`**:
> Ocorre quando o código tenta chamar um getter em uma coleção carregada como `LAZY` (ex: `instructor.getCourses()`) após o `EntityManager` / Sessão do Hibernate ter sido fechado ("*Cannot initialize proxy - no Session*").

---

## 📌 5. Mapeamento 1:1 (`@OneToOne`)

Exemplo: Um `Instructor` possui um perfil `InstructorDetail`.

### 5.1 `@OneToOne` Unidirecional

Apenas `Instructor` conhece `InstructorDetail`. A tabela `instructor` contém a Foreign Key `instructor_detail_id`.

#### SQL DDL:
```sql
CREATE TABLE instructor_detail (
    id INT NOT NULL AUTO_INCREMENT,
    youtube_channel VARCHAR(128) DEFAULT NULL,
    hobby VARCHAR(45) DEFAULT NULL,
    PRIMARY KEY (id)
);

CREATE TABLE instructor (
    id INT NOT NULL AUTO_INCREMENT,
    first_name VARCHAR(45) DEFAULT NULL,
    last_name VARCHAR(45) DEFAULT NULL,
    email VARCHAR(45) DEFAULT NULL,
    instructor_detail_id INT DEFAULT NULL,
    PRIMARY KEY (id),
    CONSTRAINT FK_DETAIL FOREIGN KEY (instructor_detail_id) 
        REFERENCES instructor_detail (id)
);
```

#### Código Entidade Owning Side (`Instructor.java`):
```java
@Entity
@Table(name="instructor")
public class Instructor {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    @Column(name="id")
    private int id;

    @Column(name="first_name")
    private String firstName;

    @Column(name="last_name")
    private String lastName;

    @Column(name="email")
    private String email;

    @OneToOne(cascade=CascadeType.ALL)
    @JoinColumn(name="instructor_detail_id")
    private InstructorDetail instructorDetail;

    // Construtores, Getters e Setters...
}
```

---

### 5.2 `@OneToOne` Bidirecional

Permite buscar o `Instructor` a partir da entidade `InstructorDetail`.

#### Código Entidade Inverse Side (`InstructorDetail.java`):
```java
@Entity
@Table(name="instructor_detail")
public class InstructorDetail {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    @Column(name="id")
    private int id;

    @Column(name="youtube_channel")
    private String youtubeChannel;

    @Column(name="hobby")
    private String hobby;

    // mappedBy aponta para o nome do ATRIBUTO na classe Instructor
    @OneToOne(mappedBy="instructorDetail", 
              cascade={CascadeType.DETACH, CascadeType.MERGE, CascadeType.PERSIST, CascadeType.REFRESH})
    private Instructor instructor;

    // Getters e Setters...
}
```

> [!TIP]
> **Deletar Apenas o Filho no Bidirecional**:
> Para deletar `InstructorDetail` sem apagar o `Instructor`, é necessário desfazer o vínculo bidirecional em memória antes de deletar:
> ```java
> tempInstructorDetail.getInstructor().setInstructorDetail(null);
> entityManager.remove(tempInstructorDetail);
> ```

---

## 📌 6. Mapeamento 1:N / N:1 (`@OneToMany` & `@ManyToOne`)

Exemplo: Um `Instructor` ministrar vários `Course`s. Cada `Course` pertence a apenas um `Instructor`.

### 6.1 `@OneToMany` Bidirecional

#### Entidade Filha Owning Side (`Course.java`):
```java
@Entity
@Table(name="course")
public class Course {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    @Column(name="id")
    private int id;

    @Column(name="title")
    private String title;

    @ManyToOne(cascade={CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFRESH})
    @JoinColumn(name="instructor_id")
    private Instructor instructor;

    // Getters e Setters...
}
```

#### Entidade Pai Inverse Side (`Instructor.java`):
```java
@Entity
@Table(name="instructor")
public class Instructor {

    // ... outros atributos

    @OneToMany(mappedBy="instructor", 
               fetch=FetchType.LAZY,
               cascade={CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFRESH})
    private List<Course> courses;

    // Método Utilitário/Conveniência para sincronização bidirecional
    public void add(Course tempCourse) {
        if (courses == null) {
            courses = new ArrayList<>();
        }
        courses.add(tempCourse);
        tempCourse.setInstructor(this); // Mantém o vínculo dos dois lados em memória
    }
}
```

---

### 6.2 Solução Elegante para LAZY Loading: JPQL com `JOIN FETCH`

Em vez de alterar globalmente o relacionamento para `FetchType.EAGER` (o que prejudica a performance geral), utiliza-se o operador **`JOIN FETCH`** no JPQL para carregar o pai e os filhos em **uma única consulta SQL customizada**.

```java
@Repository
public class AppDAOImpl implements AppDAO {

    @Autowired
    private EntityManager entityManager;

    @Override
    public Instructor findInstructorByIdJoinFetch(int theId) {
        // Carrega Instructor, seus Courses e o InstructorDetail em uma única query otimizada
        TypedQuery<Instructor> query = entityManager.createQuery(
            "select i from Instructor i " +
            "JOIN FETCH i.courses " +
            "JOIN FETCH i.instructorDetail " +
            "where i.id = :data", Instructor.class);

        query.setParameter("data", theId);

        return query.getSingleResult();
    }
}
```

---

### 6.3 `@OneToMany` Unidirecional

Exemplo: Um `Course` possui uma coleção de `Review`s. A classe `Review` **não possui** referência direta para `Course`.

#### Código Entidade `Course.java`:
```java
@Entity
@Table(name="course")
public class Course {

    // ... outros atributos

    // No Unidirecional, o @JoinColumn fica na anotação @OneToMany no lado Pai
    @OneToMany(fetch=FetchType.LAZY, cascade=CascadeType.ALL)
    @JoinColumn(name="course_id") // Aponta para a coluna FK existente na tabela review
    private List<Review> reviews;

    public void addReview(Review theReview) {
        if (reviews == null) {
            reviews = new ArrayList<>();
        }
        reviews.add(theReview);
    }
}
```

---

## 📌 7. Mapeamento N:N (`@ManyToMany`)

Exemplo: Um `Course` possui múltiplos `Student`s, e um `Student` pode se matricular em múltiplos `Course`s.

### 7.1 Estrutura do Banco de Dados (Tabela de Junção / Join Table)

Relacionamentos Muitos-para-Muitos necessitam de uma tabela intermediária (`course_student`) contendo duas Foreign Keys:

```sql
CREATE TABLE course_student (
    course_id INT NOT NULL,
    student_id INT NOT NULL,
    PRIMARY KEY (course_id, student_id),
    CONSTRAINT FK_COURSE FOREIGN KEY (course_id) REFERENCES course (id),
    CONSTRAINT FK_STUDENT FOREIGN KEY (student_id) REFERENCES student (id)
);
```

---

### 7.2 Código das Entidades (`@JoinTable`)

#### Entidade Owning Side (`Course.java`):
```java
@Entity
@Table(name="course")
public class Course {

    // ... outros atributos

    @ManyToMany(fetch=FetchType.LAZY,
                cascade={CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFRESH})
    @JoinTable(
        name="course_student",
        joinColumns=@JoinColumn(name="course_id"),         // Coluna FK que aponta para ESTA entidade (Course)
        inverseJoinColumns=@JoinColumn(name="student_id")  // Coluna FK que aponta para a OUTRA entidade (Student)
    )
    private List<Student> students;

    public void addStudent(Student theStudent) {
        if (students == null) {
            students = new ArrayList<>();
        }
        students.add(theStudent);
    }
}
```

#### Entidade Inverse Side (`Student.java`):
```java
@Entity
@Table(name="student")
public class Student {

    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    @Column(name="id")
    private int id;

    @Column(name="first_name")
    private String firstName;

    @Column(name="last_name")
    private String lastName;

    @Column(name="email")
    private String email;

    @ManyToMany(fetch=FetchType.LAZY,
                mappedBy="students", // Aponta para a propriedade 'students' na classe Course
                cascade={CascadeType.PERSIST, CascadeType.MERGE, CascadeType.DETACH, CascadeType.REFRESH})
    private List<Course> courses;

    public void addCourse(Course theCourse) {
        if (courses == null) {
            courses = new ArrayList<>();
        }
        courses.add(theCourse);
    }
}
```

---

## 📌 8. Operações de CRUD com Relacionamentos em DAO/Repository

```java
@Repository
public class AppDAOImpl implements AppDAO {

    @Autowired
    private EntityManager entityManager;

    @Override
    @Transactional
    public void save(Course theCourse) {
        // Persiste o curso e salva automaticamente dependências configuradas com PERSIST/ALL
        entityManager.persist(theCourse);
    }

    @Override
    public Course findCourseAndStudentsByCourseId(int theId) {
        TypedQuery<Course> query = entityManager.createQuery(
            "select c from Course c " +
            "JOIN FETCH c.students " +
            "where c.id = :data", Course.class);
        query.setParameter("data", theId);
        return query.getSingleResult();
    }

    @Override
    @Transactional
    public void deleteCourseById(int theId) {
        Course tempCourse = entityManager.find(Course.class, theId);
        // Remove a referência do curso da tabela de junção sem apagar os alunos do banco
        entityManager.remove(tempCourse);
    }

    @Override
    @Transactional
    public void deleteStudentById(int theId) {
        Student tempStudent = entityManager.find(Student.class, theId);
        // Remove o registro do aluno e desvincula dos cursos sem apagar os cursos do banco
        entityManager.remove(tempStudent);
    }
}
```

---

## 📋 Tabela Resumo das Anotações JPA Avançadas

| Anotação | Local de Uso | Propósito / Função |
| :--- | :--- | :--- |
| **`@OneToOne`** | Atributo | Mapeia relacionamento de 1 para 1 entre duas entidades. |
| **`@OneToMany`** | Coleção (`List`) | Mapeia relacionamento de 1 para Muitos (ex: Um Instrutor tem vários Cursos). |
| **`@ManyToOne`** | Atributo | Mapeia relacionamento de Muitos para 1 (ex: Vários Cursos pertencem a um Instrutor). |
| **`@ManyToMany`** | Coleção (`List`) | Mapeia relacionamento Muitos para Muitos (ex: Cursos e Alunos). |
| **`@JoinColumn(name="fk_id")`** | Atributo/Coleção | Especifica o nome da coluna de Foreign Key (FK) na tabela do banco de dados. |
| **`@JoinTable(...)`** | Coleção | Define o nome da tabela de junção e as colunas de FK para relacionamentos `@ManyToMany`. |
| **`mappedBy="propriedade"`**| Atributos em anotações de relacionamento | Declara o lado inverso da relação, indicando qual atributo no outro lado é o proprietário. |
| **`fetch = FetchType.LAZY`**| Anotações de relacionamento | Carrega dados da associação sob demanda (recomendado por questões de performance). |
| **`fetch = FetchType.EAGER`**| Anotações de relacionamento | Carrega dados associados imediatamente junto com a entidade principal. |
| **`JOIN FETCH`** | Consultas JPQL | Carrega relacionamentos configurados como `LAZY` em uma única query SQL otimizada. |
