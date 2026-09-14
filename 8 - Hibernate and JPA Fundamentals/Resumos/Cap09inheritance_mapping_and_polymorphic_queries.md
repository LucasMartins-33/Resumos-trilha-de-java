# Capítulo 09: Inheritance Mapping and Polymorphic Queries

## Introdução
O modelo relacional de banco de dados não possui o conceito nativo de "herança" como a orientação a objetos. Portanto, para salvar uma hierarquia de classes do Java no banco, precisamos realizar um Mapeamento de Herança (*Inheritance Mapping*). Ao fazermos isso, ganhamos também o poder das **Consultas Polimórficas** (*Polymorphic Queries*), onde podemos consultar pela superclasse e o JPA traz as subclasses correspondentes automaticamente.

Neste capítulo, estudaremos as três estratégias de mapeamento de herança da JPA, a anotação `@MappedSuperclass` e duas novidades interessantes introduzidas no Hibernate 6.6: a Herança de Embeddables e a anotação `@ConcreteProxy`.

---

## 1. Mapeando Herança de Entidades

A estratégia de herança é definida na superclasse, usando a anotação `@Inheritance`. Existem 3 estratégias fundamentais:

### 1.1. Single Table Strategy (`InheritanceType.SINGLE_TABLE`)
Esta é a estratégia **padrão** da JPA. Ela agrupa os dados de toda a hierarquia de classes em uma **única tabela**. 

Para diferenciar qual linha pertence a qual classe filha (ex: `Dog` ou `Cat`), o JPA cria automaticamente uma coluna discriminadora no banco de dados chamada `DTYPE` (*Discriminator Type*).

**Exemplo de Código:**
```java
@Entity
@Inheritance(strategy=InheritanceType.SINGLE_TABLE)
public abstract class Animal {
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY) 
    private Long id;
    
    private String name;
    //...
}

@Entity
public class Dog extends Animal {
    private String breed;
}

@Entity
public class Cat extends Animal {
    private Integer purringLevel;
}
```

* **Vantagem:** A **performance das consultas polimórficas é excelente**. Se você fizer `SELECT a FROM Animal a`, o Hibernate emite um simples `SELECT * FROM animal` no banco, sem a necessidade de *JOINs*.
* **Desvantagem:** Integridade relacional. Como os campos exclusivos das classes filhas (`breed`, `purringLevel`) vivem na mesma tabela, eles ficarão nulos para as linhas dos outros tipos. Portanto, você **não pode adicionar a constraint `NOT NULL`** nos campos de uma subclasse.

### 1.2. Joined Strategy (`InheritanceType.JOINED`)
Nesta estratégia, teremos uma tabela para a superclasse (ex: `Animal`) e uma tabela para cada subclasse (ex: `Dog` e `Cat`).
As tabelas filhas guardarão **apenas as colunas exclusivas delas** (não herdadas). A Chave Primária das tabelas filhas funciona simultaneamente como Chave Estrangeira (*Foreign Key*) apontando para o registro na tabela pai.

* **Vantagem:** O banco fica altamente normalizado. Você pode (e deve) aplicar validações como `NOT NULL` nos atributos das subclasses, pois eles pertencem a tabelas exclusivas.
* **Desvantagem:** Péssima performance em consultas polimórficas. Para buscar um `Animal` de qualquer tipo, o Hibernate terá que emitir uma enxurrada de `LEFT OUTER JOINs` com **todas as tabelas filhas conhecidas**, além de uma complexa cláusula `CASE` SQL para inferir o tipo em tempo de execução.

### 1.3. Table Per Class Strategy (`InheritanceType.TABLE_PER_CLASS`)
(Tabela por Classe Concreta). Se `Animal` for abstrata, o banco não terá uma tabela `Animal`. Terá apenas as tabelas `Dog` e `Cat`.
Cada uma dessas tabelas armazena **todos** os atributos (tanto os próprios quanto os herdados).
Nesta estratégia, as tabelas não possuem relacionamento de *Foreign Key* entre si.

* **Atenção aos IDs:** Como os IDs devem ser exclusivos em todas as tabelas derivadas (para não colidir), não podemos usar a estratégia de auto-incremento simples. Recomenda-se usar a estratégia `TABLE` (`GenerationType.TABLE`) na geração do ID.
* **Consultas Polimórficas:** Para consultar por `Animal`, o banco terá que executar um imenso `UNION` entre todas as tabelas (juntando o `SELECT` de `Dog` e de `Cat`). O desempenho costuma ser pior do que a estratégia JOINED e ela foi até taxada como "Opcional" na especificação oficial do JPA.

---

## 2. A anotação `@MappedSuperclass`

Diferente de `@Inheritance`, uma classe anotada com `@MappedSuperclass` **não é uma entidade JPA** e não possui tabela no banco de dados. Ela atua puramente no mundo Orientado a Objetos (memória/Java).

Ela serve como uma classe base comum, cujos campos (como `id`, `owner`, `createdAt`) serão "copiados" para dentro das tabelas das entidades filhas (`CreditAccount`, `DebitAccount`).

**Exemplo:**
```java
@MappedSuperclass
public class Account {
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;
    private String owner;
    private BigDecimal balance;
}

@Entity
@Table(name="debit_account")
public class DebitAccount extends Account {
    private BigDecimal overdraftFee;
}
```
**Peculiaridade:** Por não ser uma entidade, **não é possível fazer consultas polimórficas** em cima dela. Tentar executar `SELECT a FROM Account a` lançará uma exceção (Exception).

---

## 3. Embeddable Inheritance (A partir do Hibernate 6.6)

Até o Hibernate 6.6, a herança era suportada apenas em classes `@Entity`. Agora, também é suportada em objetos embutíveis (`@Embeddable`).
Uma classe pode ter um campo de um tipo `@Embeddable` (`Animal`) e instanciar versões específicas dele (`Fish` ou `Dog`).

```java
@Embeddable
public class Animal { private String name; }

@Embeddable
public class Fish extends Animal { private int fins; }

@Entity
public class Owner {
    @Id private Long id;
    @Embedded private Animal pet;
}
```
Internamente, essa funcionalidade age quase como a estratégia `SINGLE_TABLE`. Todos os atributos herdados e das subclasses (`name`, `fins`) são achatados como colunas dentro da tabela de `Owner`. Para que o Hibernate saiba de qual tipo concreto é o objeto para reinstanciá-lo, ele adiciona automaticamente uma coluna discriminadora no banco de dados, chamada `pet_dtype`.

---

## 4. O Problema do Lazy Loading na Herança e o `@ConcreteProxy`

Uma das grandes armadilhas no JPA ocorre ao misturar relacionamentos preguiçosos (`FetchType.LAZY`) com a Herança.
Imagine a relação `ManyToOne` entre um Dono (`Owner`) e o seu Pet (`Animal`), onde Animal tem as filhas concretas `Dog` e `Fish`.

```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "animal_id")
private Animal animal;
```

**O Problema:** 
Ao carregar o `Owner` usando `em.find(Owner.class, 1L)`, o Hibernate vê a relação LAZY. Para evitar carregar o `Animal` do banco, ele gera um **Objeto Proxy Falso** do tipo base (`Animal`).
Quando você tenta checar se esse animal na verdade era um peixe:
```java
Animal animal = owner.getAnimal();
boolean isFish = animal instanceof Fish; // Retornará FALSE!!!
int fins = ((Fish) animal).getFins(); // ClassCastException!!!
```
A JVM vê que o proxy é da classe `Animal` e impede a conversão (Cast), quebrando a lógica e lançando um erro.

**A Solução (Hibernate 6.6):**
Para consertar isso, adicione a anotação **`@org.hibernate.annotations.ConcreteProxy`** no topo da superclasse (raiz da herança):

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@ConcreteProxy // Solução do Hibernate 6.6
public class Animal { ... }
```

Ao fazer isso, quando buscar o `Owner`, o Hibernate fará um leve `LEFT JOIN` adicional exclusivamente para descobrir qual é o valor da coluna `DTYPE` do animal apontado. Com esse dado na mão, ele instanciará o **Proxy da Classe Certa** (um Proxy de `Fish`), fazendo com que as operações de `instanceof` e *Cast* funcionem perfeitamente.
