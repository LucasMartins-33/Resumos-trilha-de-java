# Questões Teóricas - Capítulo 03: Hibernate / JPA CRUD

### Questão 1
Qual é a relação entre **JPA (Jakarta Persistence API)** e **Hibernate**?

> [!faq]- Resposta
> O JPA é a especificação padrão do Java que define interfaces e regras de mapeamento objeto-relacional (ORM). O Hibernate é a implementação concreta dessa especificação, fornecendo a engine real que executa o código SQL e gerencia a persistência no banco de dados.

---

### Questão 2
Para que servem as anotações `@Entity`, `@Table`, `@Id` e `@GeneratedValue` no mapeamento de uma entidade JPA?

> [!faq]- Resposta
> * **`@Entity`**: Marca a classe Java como uma tabela persistente gerenciada pelo JPA.
> * **`@Table(name="...")`**: Especifica o nome exato da tabela no banco de dados relacional.
> * **`@Id`**: Define o atributo que atua como Chave Primária (Primary Key) da entidade.
> * **`@GeneratedValue(strategy = GenerationType.IDENTITY)`**: Define a estratégia de geração automática da Chave Primária (ex: autoincremento no MySQL).

---

### Questão 3
Qual é o papel da interface `EntityManager` no Hibernate/JPA?

> [!faq]- Resposta
> O `EntityManager` é a interface principal utilizada para interagir com o contexto de persistência. Ele fornece métodos nativos para operações de CRUD, tais como `persist()` (inserir), `find()` (buscar por ID), `merge()` (atualizar) e `remove()` (excluir), além de criar consultas em JPQL.

---

### Questão 4
O que é o padrão **DAO (Data Access Object)** e qual a função da anotação `@Repository` em sua implementação no Spring Boot?

> [!faq]- Resposta
> O padrão DAO isola a lógica de persistência e acesso a dados da camada de negócios da aplicação. A anotação `@Repository` marca a classe DAO como um componente Spring de persistência e ativa a tradução de exceções JDBC/SQL nativas em exceções não-checadas do Spring (`DataAccessException`).

---

### Questão 5
Para que serve a anotação `@Transactional` e qual camada da aplicação deve ser responsável por declará-la?

> [!faq]- Resposta
> A anotação `@Transactional` gerencia as transações do banco de dados automaticamente (iniciando, realizando `commit` em caso de sucesso ou `rollback` em caso de exceções não-checadas). Ela deve ser declarada no método da camada DAO ou de Serviço que realiza operações de gravação, alteração ou exclusão no banco de dados.

---

### Questão 6
O que é **JPQL (Java Persistence Query Language)** e em que ela difere do SQL nativo?

> [!faq]- Resposta
> JPQL é uma linguagem de consulta orientada a objetos do JPA. Em vez de fazer consultas referenciando colunas e tabelas nativas do banco de dados (como no SQL), o JPQL faz consultas referenciando as **Classes Entidades** e seus **Atributos Java** (ex: `SELECT s FROM Student s WHERE s.lastName = :theData`).

---

### Questão 7
Qual é a função do parâmetro nomeado (ex: `:theData`) em consultas JPQL e por que ele deve ser utilizado em vez da concatenação de strings?

> [!faq]- Resposta
> Parâmetros nomeados evitam ataques de **SQL Injection** ao sanitizar e tratar adequadamente os dados passados pelo usuário. Eles também permitem que o provedor JPA reutilize planos de execução de consultas previamente compilados.

---

### Questão 8
O que é a interface `CommandLineRunner` do Spring Boot e para que ela é comumente utilizada em aplicações de teste?

> [!faq]- Resposta
> O `CommandLineRunner` é uma interface funcional do Spring Boot que fornece um método `run(String... args)` executado automaticamente logo após o contexto da aplicação e todos os Beans Spring serem carregados. É comumente usada para rodar scripts de teste no console ou popular o banco de dados inicial.

---

### Questão 9
Como funciona a atualização de um objeto via `EntityManager.merge()` no JPA?

> [!faq]- Resposta
> O método `merge()` recebe um objeto desanexado (*detached*) do contexto de persistência, pesquisa no banco a entidade correspondente pelo ID, atualiza seus atributos com os novos valores passados e re-anexa a entidade no contexto gerenciado, executando um comando `UPDATE` no banco de dados ao realizar o commit.

---

### Questão 10
Como realizar uma exclusão em lote no banco de dados utilizando JPQL com o `EntityManager`?

> [!faq]- Resposta
> Utiliza-se o método `entityManager.createQuery("DELETE FROM Student WHERE lastName = :theLastName")`, associa-se o parâmetro com `.setParameter()` e executa-se a instrução através do método `.executeUpdate()`, que retorna o número de registros deletados.
