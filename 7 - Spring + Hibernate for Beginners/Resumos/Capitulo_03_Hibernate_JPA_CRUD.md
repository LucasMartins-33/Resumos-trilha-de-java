# Capítulo 03: Hibernate / JPA CRUD

Este resumo detalha o acesso a banco de dados relacional com **Spring Boot 4**, **Jakarta Persistence API (JPA)** e **Hibernate**. Ele cobre a arquitetura ORM, padrões DAO, criação de Entidades, operações fundamentais de CRUD (Create, Read, Update, Delete) com `EntityManager` e **JPQL**, além do gerenciamento de transações, chaves primárias e geração automática de esquemas DDL.

---

## 📌 1. Visão Geral de ORM, Hibernate, JPA e JDBC

### O que é ORM (Object-Relational Mapping)?
Técnica para mapear objetos de uma linguagem orientada a objetos (Java) para tabelas de um banco de dados relacional (SQL).

### Relação entre as Tecnologias

```text
+-------------------------------------------------------------+
|                     Minha Aplicação Java                    |
+-------------------------------------------------------------+
|   JPA (Jakarta Persistence API) -> Especificação / Interfaces|
+-------------------------------------------------------------+
|   Hibernate Framework           -> Implementação / Provedor |
+-------------------------------------------------------------+
|   JDBC Driver (MySQL Driver)    -> Comunicação de Baixo Nível|
+-------------------------------------------------------------+
|                     Banco de Dados MySQL                    |
+-------------------------------------------------------------+
```

* **JPA (Jakarta Persistence API)**: É a **especificação padrão** do Java para ORM (somente interfaces e anotações). Evita *vendor lock-in* (preso a um único fornecedor).
* **Hibernate**: É a **implementação principal e padrão** do JPA no Spring Boot. Ele realiza o trabalho pesado de conversão de objetos Java em comandos SQL.
* **JDBC**: O Hibernate utiliza o JDBC por trás dos panos para se comunicar diretamente com o banco de dados.

---

## 📌 2. Configuração do Projeto e Banco de Dados (MySQL)

### Propriedades de Conexão no `src/main/resources/application.properties`

```properties
# Configuração da Conexão JDBC
spring.datasource.url=jdbc:mysql://localhost:3306/student_tracker?useSSL=false&serverTimezone=UTC
spring.datasource.username=springstudent
spring.datasource.password=springstudent

# Desativar a Banner do Spring Boot (Opcional para CommandLineRunner)
spring.main.banner-mode=off

# Configuração do Nível de Log
logging.level.root=warn
logging.level.org.hibernate.SQL=debug
logging.level.org.hibernate.orm.jdbc.bind=trace
```

> [!NOTE]
> **Detecção Automática de Driver**: No Spring Boot 3 / 4, a declaração de `spring.datasource.driver-class-name` é opcional, pois o Spring Boot detecta a classe do driver MySQL automaticamente através do prefixo da URL `jdbc:mysql://`.

---

## 📌 3. Mapeamento de Entidade JPA (`@Entity`)

Uma **Entidade** é uma classe Java anotada que representa uma tabela no banco de dados.

### Requisitos Obrigatórios de uma Entidade JPA:
1. Ter a anotação `@Entity`.
2. Ter um **construtor público ou protegido sem argumentos (`no-arg constructor`)**.
3. Possuir um campo anotado como chave primária (`@Id`).

### Anotações Mapeadas no Código Java

```java
package com.luv2code.cruddemo.entity;

import jakarta.persistence.*;

@Entity
@Table(name = "student") // Mapeia a classe para a tabela "student"
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY) // Auto-incremento no MySQL
    @Column(name = "id")
    private int id;

    @Column(name = "first_name")
    private String firstName;

    @Column(name = "last_name")
    private String lastName;

    @Column(name = "email")
    private String email;

    // 1. Construtor sem argumentos (Obrigatório para o JPA/Hibernate)
    public Student() { }

    // 2. Construtor utilitário (sem o ID, já que o ID é gerado pelo banco)
    public Student(String firstName, String lastName, String email) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
    }

    // Getters e Setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getFirstName() { return firstName; }
    public void setFirstName(String firstName) { this.firstName = firstName; }

    public String getLastName() { return lastName; }
    public void setLastName(String lastName) { this.lastName = lastName; }

    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }

    // toString() para facilidade de depuração
    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", firstName='" + firstName + '\'' +
                ", lastName='" + lastName + '\'' +
                ", email='" + email + '\'' +
                '}';
    }
}
```

### Estratégias de Geração de Chave Primária (`GenerationType`)
* **`GenerationType.IDENTITY`**: Utiliza a coluna auto-incremento nativa do banco de dados (recomendado para MySQL).
* **`GenerationType.SEQUENCE`**: Utiliza uma sequência do banco de dados (comum em PostgreSQL e Oracle).
* **`GenerationType.TABLE`**: Utiliza uma tabela genérica auxiliar para gerar chaves únicas.
* **`GenerationType.AUTO`**: Deixa o provedor JPA escolher a melhor estratégia automaticamente.
* **`GenerationType.UUID`**: Gera um identificador único universal (UUID de 128 bits).

---

## 📌 4. O Padrão Data Access Object (DAO) com EntityManager

O **EntityManager** é a interface principal do JPA para realizar operações de persistência (salvar, buscar, atualizar, deletar).

### Anotação `@Repository`
* Aplicada nas classes de implementação DAO.
* É uma especialização de `@Component` que habilita o escaneamento de componentes.
* Converte exceções JDBC checadas (*checked exceptions*) em exceções não checadas do Spring (`DataAccessException`).

### Anotação `@Transactional`
* Gerencia o início e término automático das transações no banco de dados.
* **Obrigatória** para operações de alteração de dados (**Create, Update, Delete**).
* **Desnecessária** para operações puras de leitura/consulta.

### 1. Interface DAO (`StudentDAO.java`)
```java
package com.luv2code.cruddemo.dao;

import com.luv2code.cruddemo.entity.Student;
import java.util.List;

public interface StudentDAO {
    void save(Student theStudent);
    Student findById(Integer id);
    List<Student> findAll();
    List<Student> findByLastName(String theLastName);
    void update(Student theStudent);
    void delete(Integer id);
    int deleteAll();
}
```

### 2. Implementação DAO (`StudentDAOImpl.java`)
```java
package com.luv2code.cruddemo.dao;

import com.luv2code.cruddemo.entity.Student;
import jakarta.persistence.EntityManager;
import jakarta.persistence.TypedQuery;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Repository
public class StudentDAOImpl implements StudentDAO {

    private final EntityManager entityManager;

    // Injeção de dependência por construtor do EntityManager do JPA
    @Autowired
    public StudentDAOImpl(EntityManager entityManager) {
        this.entityManager = entityManager;
    }

    // CREATE (Salvar)
    @Override
    @Transactional
    public void save(Student theStudent) {
        entityManager.persist(theStudent);
    }

    // READ (Buscar por ID)
    @Override
    public Student findById(Integer id) {
        return entityManager.find(Student.class, id);
    }

    // QUERY (Buscar Todos)
    @Override
    public List<Student> findAll() {
        // JPQL: Utiliza o nome da Classe Java 'Student', não o nome da tabela 'student'
        TypedQuery<Student> theQuery = entityManager.createQuery("FROM Student ORDER BY lastName ASC", Student.class);
        return theQuery.getResultList();
    }

    // QUERY (Buscar com Parâmetro Nomeado)
    @Override
    public List<Student> findByLastName(String theLastName) {
        // :theData é um parâmetro nomeado no JPQL
        TypedQuery<Student> theQuery = entityManager.createQuery(
                "FROM Student WHERE lastName = :theData", Student.class);
        
        theQuery.setParameter("theData", theLastName);
        return theQuery.getResultList();
    }

    // UPDATE (Atualizar)
    @Override
    @Transactional
    public void update(Student theStudent) {
        entityManager.merge(theStudent);
    }

    // DELETE (Remover Único)
    @Override
    @Transactional
    public void delete(Integer id) {
        Student theStudent = entityManager.find(Student.class, id);
        if (theStudent != null) {
            entityManager.remove(theStudent);
        }
    }

    // DELETE (Remover Todos via JPQL)
    @Override
    @Transactional
    public int deleteAll() {
        int numRowsDeleted = entityManager.createQuery("DELETE FROM Student").executeUpdate();
        return numRowsDeleted;
    }
}
```

---

## 📌 5. Consulta com JPQL (Jakarta Persistence Query Language)

O **JPQL** é a linguagem de consulta do JPA baseada nos nomes das **Entidades Java e seus campos**, e **não nas tabelas e colunas** do banco de dados.

### Sintaxe Estrita (Strict JPQL) vs Sintaxe Simplificada (HQL)
* **Estrita (Com SELECT)**: `SELECT s FROM Student s WHERE s.lastName = :theData`
* **Simplificada**: `FROM Student WHERE lastName = :theData`

### Exemplos Comuns de JPQL:

```java
// 1. Seleção completa com ordenação decrescente
TypedQuery<Student> q1 = entityManager.createQuery("FROM Student ORDER BY lastName DESC", Student.class);

// 2. Cláusula WHERE com múltiplos predicados (OR)
TypedQuery<Student> q2 = entityManager.createQuery(
    "FROM Student WHERE lastName = 'Doe' OR firstName = 'Daffy'", Student.class);

// 3. Operador LIKE (Busca por padrão no e-mail)
TypedQuery<Student> q3 = entityManager.createQuery(
    "FROM Student WHERE email LIKE '%@luv2code.com'", Student.class);
```

---

## 📌 6. Executando Operações no `CommandLineRunner`

O `CommandLineRunner` é uma interface do Spring Boot para executar códigos logo após o carregamento dos Beans na inicialização da aplicação.

```java
package com.luv2code.cruddemo;

import com.luv2code.cruddemo.dao.StudentDAO;
import com.luv2code.cruddemo.entity.Student;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;

import java.util.List;

@SpringBootApplication
public class CruddemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(CruddemoApplication.class, args);
    }

    @Bean
    public CommandLineRunner commandLineRunner(StudentDAO studentDAO) {
        return runner -> {
            // Escolha um método para testar a operação CRUD:
            createStudent(studentDAO);
            // readStudent(studentDAO);
            // queryForStudents(studentDAO);
            // updateStudent(studentDAO);
            // deleteStudent(studentDAO);
        };
    }

    private void createStudent(StudentDAO studentDAO) {
        System.out.println("Criando novo estudante...");
        Student tempStudent = new Student("Paul", "Doe", "paul@luv2code.com");

        System.out.println("Salvando o estudante...");
        studentDAO.save(tempStudent);

        System.out.println("Estudante salvo. ID gerado: " + tempStudent.getId());
    }

    private void readStudent(StudentDAO studentDAO) {
        System.out.println("Buscando estudante com ID: 1");
        Student myStudent = studentDAO.findById(1);
        System.out.println("Estudante encontrado: " + myStudent);
    }

    private void updateStudent(StudentDAO studentDAO) {
        int studentId = 1;
        System.out.println("Buscando estudante com ID: " + studentId);
        Student myStudent = studentDAO.findById(studentId);

        System.out.println("Atualizando nome para 'Scooby'...");
        myStudent.setFirstName("Scooby");
        studentDAO.update(myStudent);

        System.out.println("Estudante atualizado: " + myStudent);
    }

    private void deleteStudent(StudentDAO studentDAO) {
        int studentId = 3;
        System.out.println("Deletando estudante com ID: " + studentId);
        studentDAO.delete(studentId);
    }
}
```

---

## 📌 7. Geração Automática de Tabelas (`ddl-auto`)

O Hibernate pode criar ou modificar as tabelas do banco de dados automaticamente com base nas anotações das entidades Java.

### Propriedade no `application.properties`:
```properties
spring.jpa.hibernate.ddl-auto=create
```

### Opções do `ddl-auto`:

| Valor | Comportamento | Uso Recomendado |
| :--- | :--- | :--- |
| **`none`** | Nenhuma ação DDL é executada. | Produção / Projetos Corporativos. |
| **`create`** | **Apaga (`DROP`) as tabelas existentes** e cria novas a cada inicialização da aplicação (Perde dados!). | Testes rápidos / Desenvolvimento inicial. |
| **`create-drop`** | Cria as tabelas na inicialização e as apaga ao encerrar o sistema. | Testes Unitários integrados. |
| **`update`** | Atualiza o esquema da tabela adicionando novas colunas sem apagar os dados existentes. | Desenvolvimento Local de pequenos projetos. |
| **`validate`** | Valida se as tabelas do banco correspondem exatamente às Entidades anotadas. | Ambientes de homologação. |

> [!CAUTION]
> **Alerta Vermelho para Ambientes de Produção**:
> NUNCA utilize `create`, `create-drop` ou `update` em bancos de dados de **Produção**! Em projetos reais, utilize **scripts SQL gerenciados** por ferramentas de migração de esquema como **Flyway** ou **Liquibase**.

---

## 📌 8. Comparativo: `EntityManager` vs `JpaRepository`

| Recurso | `EntityManager` (Visto no Cap 03) | `JpaRepository` (Veremos nos próx. capítulos) |
| :--- | :--- | :--- |
| **Nível de Abstração** | Baixo nível (Controle granular) | Alto nível (Métodos CRUD prontos por interface) |
| **Controle de Queries** | Ideal para queries JPQL complexas, SQL nativo e Stored Procedures | Métodos prontos (`findAll()`, `save()`, etc.) e *Query Methods* |
| **Flexibilidade** | Alta flexibilidade para manipulação de contextos JPA | Alta produtividade para CRUDs simples |

---

## 📋 Tabela Resumo dos Métodos do `EntityManager`

| Operação CRUD | Método JPA / JPQL | Requer `@Transactional`? | Descrição |
| :--- | :--- | :---: | :--- |
| **Create** | `entityManager.persist(entity)` | **Sim** | Insere um novo objeto no banco. |
| **Read (por ID)** | `entityManager.find(Class, id)` | Não | Busca o objeto pela chave primária (retorna `null` se não achar). |
| **Read (Lista)** | `entityManager.createQuery(jpql, Class)` | Não | Executa uma consulta em linguagem JPQL. |
| **Update** | `entityManager.merge(entity)` | **Sim** | Atualiza um objeto existente ou insere se não existir. |
| **Delete** | `entityManager.remove(entity)` | **Sim** | Remove o objeto informado do banco. |
| **Execute DML** | `query.executeUpdate()` | **Sim** | Executa queries de alteração em lote (`UPDATE` ou `DELETE` via JPQL). |
