# Capítulo 11: Logic in Database

## Introdução
Muitas vezes, a nossa aplicação interage com volumes massivos de dados. Em certas ocasiões, trazer todos esses dados para a memória da aplicação Java (JPA) apenas para realizar cálculos e agregações pode ser um desastre de performance, gerando *gargalos* (*bottlenecks*) e excesso de viagens pela rede (*network round-trips*). 

A solução ideal nestes cenários pesados é delegar a lógica para o próprio Banco de Dados através de **Stored Procedures**, **Database Functions** e **Database Views**. Neste capítulo, aprenderemos como integrar esses poderosos recursos nativos dos bancos de dados diretamente com a JPA/Hibernate.

---

## 1. Stored Procedures (Procedimentos Armazenados)

Uma *Stored Procedure* é um bloco de código SQL salvo no banco que executa uma tarefa específica. Ela pode receber parâmetros de entrada (`IN`), devolver parâmetros de saída (`OUT`) e até mesmo retornar *Result Sets* (conjuntos de linhas, como um `SELECT`).

### 1.1. Invocando Stored Procedures Escalares
Vamos supor que criamos uma Procedure no banco de dados chamada `count_employee_by_department` que recebe o nome de um departamento e devolve a quantidade total de empregados nele. 

**Passo 1: O Mapeamento na Entidade**
Na nossa entidade, usamos a anotação `@NamedStoredProcedureQuery` (frequentemente agrupadas dentro de `@NamedStoredProcedureQueries`) para mapear o procedimento. 

```java
@Entity
@NamedStoredProcedureQueries({
    @NamedStoredProcedureQuery(
        name = "CountByDepartmentProcedure",           // Nome usado pelo JPA
        procedureName = "count_employee_by_department", // Nome real no banco
        parameters = {
            @StoredProcedureParameter(name = "dept", type = String.class, mode = ParameterMode.IN),
            @StoredProcedureParameter(name = "count", type = Integer.class, mode = ParameterMode.OUT)
        }
    )
})
public class Employee { ... }
```

**Passo 2: Invocando pelo EntityManager**
No lado do cliente, a execução é limpa e direta:
```java
StoredProcedureQuery query = em.createNamedStoredProcedureQuery("CountByDepartmentProcedure");
query.setParameter("dept", "Engineering");
query.execute(); // Chama o procedimento no banco

Integer count = (Integer) query.getOutputParameterValue("count");
System.out.println("Total de Empregados: " + count);
```

### 1.2. Retornando Entidades
Se a Procedure executar um `SELECT * FROM employee`, nós podemos pedir para o JPA mapear essas linhas diretamente para a nossa classe Entidade. Para isso, basta adicionar a propriedade `resultClasses` no mapeamento:

```java
@NamedStoredProcedureQuery(
    name = "FindByDepartmentProcedure",
    procedureName = "find_employee_by_department",
    resultClasses = Employee.class, // <-- O segredo está aqui!
    parameters = {
        @StoredProcedureParameter(name = "dept", type = String.class, mode = ParameterMode.IN)
    }
)
```
E a captura na aplicação é feita como em queries normais: `List<Employee> list = query.getResultList();`.

### 1.3. Retornando Colunas Parciais via DTOs (Data Transfer Objects)
E se a Procedure retornar apenas algumas colunas (`SELECT name, salary`) ao invés da entidade completa?
O JPA devolverá uma incômoda lista de array de objetos (`List<Object[]>`), o que tira toda a tipagem forte do Java. 

A solução elegante é mapear o retorno da procedure para um objeto **DTO (Data Transfer Object)**. Isso é feito combinando `@SqlResultSetMapping` e `@ConstructorResult`.

**A classe DTO:** (Um POJO puro, sem anotações JPA)
```java
public class EmployeeDto {
    private String name;
    private Integer salary;
    public EmployeeDto(String name, Integer salary) { ... }
}
```

**O Mapeamento (Fica na entidade `Employee`):**
```java
@SqlResultSetMapping(
    name = "EmployeeDtoMapping", // Nome do mapeamento
    classes = @ConstructorResult(
        targetClass = EmployeeDto.class, // Classe de destino
        columns = {
            @ColumnResult(name = "name"),
            @ColumnResult(name = "salary")
        }
    )
)
@NamedStoredProcedureQuery(
    name = "FindNameAndSalaryByDepartmentProcedure",
    procedureName = "find_name_and_salary_by_department",
    resultSetMappings = "EmployeeDtoMapping", // <-- Amarração feita aqui!
    parameters = {
        @StoredProcedureParameter(name = "dept", type = String.class, mode = ParameterMode.IN)
    }
)
```
Dessa forma, o `query.getResultList()` devolverá um lindo e tipado `List<EmployeeDto>`.

---

## 2. Database Functions (Funções de Banco de Dados)

Diferente das Stored Procedures (que executam lógicas complexas e não precisam retornar nada), as **Database Functions** são obrigadas a retornar **um único valor**. São excelentes para efetuar cálculos no banco, como por exemplo: `calculate_bonus(empId, bonusPct)`.

### 2.1. Execução Nativa
Podemos chamar funções executando Queries nativas simples:
```java
Query query = em.createNativeQuery("SELECT calculate_bonus(:empId, :bonusPct)");
query.setParameter("empId", 2L);
query.setParameter("bonusPct", 10);
Double bonus = (Double) query.getSingleResult();
```

### 2.2. Execução através da JPQL
A mágica acontece quando integramos funções nativas do banco diretamente em consultas JPQL através da palavra reservada `FUNCTION`. 
O Hibernate saberá que precisa traduzir essa palavra para a chamada nativa da função!

**Exemplo no SELECT:**
```java
Query q = em.createQuery("SELECT FUNCTION('calculate_bonus', e.id, :bonusPct) FROM Employee e WHERE e.id = :empId");
```

**Exemplo no WHERE:** (Traz todos que ganham menos do que o bônus de um funcionário específico)
```java
Query q = em.createQuery(
    "SELECT count(e) FROM Employee e WHERE e.salary <= FUNCTION('calculate_bonus', :empId, :bonusPct)"
);
```

---

## 3. Database Views (Visões)

Uma View no banco de dados é uma "tabela virtual" criada a partir de uma consulta pré-definida. Elas são geniais para consultas altamente focadas em leitura (*Read-Only*), como agregação de dados para **Dashboards** (ex: quantidade de pedidos e valor total gasto agrupado por cliente).

### 3.1. Mapeando Views no JPA
No JPA, **uma View é mapeada da exata mesma forma que uma Tabela**. Nós criamos uma classe anotada com `@Entity`.

O grande pulo do gato ao mapear Views é o uso da anotação `@Immutable` do Hibernate. 

**Como as Views, em 99% dos casos, agregam dados de múltiplas tabelas (com JOIN e GROUP BY), elas não aceitam comandos de INSERT ou UPDATE.** Ao anotarmos a classe com `@Immutable`, nós avisamos ao Hibernate: *"Esta entidade serve apenas para leitura. Não monitore estado sujo (dirty checking) e nunca tente enviar comandos UPDATE para ela no banco!"*

```java
import org.hibernate.annotations.Immutable; // Atenção à importação

@Entity
@Immutable // Proteção vital para Views!
@Table(name = "customer_order_summary")
public class CustomerOrderSummary {

    @Id
    @Column(name = "customer_id")
    private Long customerId;

    @Column(name = "customer_name")
    private String customerName;

    @Column(name = "total_orders")
    private Long totalOrders;

    @Column(name = "total_spent")
    private BigDecimal totalSpent;
    
    // Getters omitidos...
}
```
A partir desse momento, basta fazer uma consulta JPQL tradicional para carregar os dados agregados da View com máxima performance:
```java
List<CustomerOrderSummary> dashboardData = em.createQuery(
    "SELECT cos FROM CustomerOrderSummary cos", CustomerOrderSummary.class
).getResultList();
```
