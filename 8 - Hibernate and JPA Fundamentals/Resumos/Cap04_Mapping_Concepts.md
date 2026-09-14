# Capítulo 04: Mapping Concepts (Guia Definitivo)

Neste capítulo, elevamos o nível. Deixamos de mapear apenas classes isoladas e passamos a estudar como mapear **relacionamentos complexos** entre objetos do mundo real (POJOs) para tabelas de um banco de dados relacional. 

Este documento foi elaborado para servir como um **guia completo e detalhado**. Ele não apenas apresenta os códigos, mas explica o **"porquê"** de cada anotação e regra do Hibernate.

---

## 1. Agregação vs Composição e Tipos de Valor
*(Aulas 20 e 21)*

Antes de escrever qualquer anotação no Java, precisamos entender como os objetos se relacionam conceitualmente. Na orientação a objetos, os relacionamentos "Todo-Parte" se dividem em dois níveis de força:

* **Agregação (Aggregation):** É uma relação mais fraca. A "Parte" pode existir sem o "Todo". 
  * *Exemplo:* Uma Banda (Todo) e seus Artistas (Partes). Se a banda acabar, os artistas não morrem; eles vão para outras bandas. O ciclo de vida deles é independente.
* **Composição (Composition):** É uma relação forte, de total dependência. Se o "Todo" for destruído, as "Partes" são destruídas junto. Além disso, as Partes não são compartilhadas. 
  * *Exemplo:* Uma Casa (Todo) e seus Cômodos (Partes). Se a casa for demolida, o quarto deixa de existir. O seu quarto não pode pertencer à casa do vizinho simultaneamente.

### Entidades vs Tipos de Valor (Value Types)
Ao mapear nosso sistema para o banco de dados, devemos fazer uma pergunta crucial para cada classe: **"A identidade deste objeto importa isoladamente para o meu banco de dados?"**

1. **Entity Type (Entidade):** São objetos que têm vida própria. Eles precisam ser buscados independentemente no banco de dados (ex: `User`, `Product`, `Student`). Eles recebem a anotação `@Entity` e **obrigatoriamente possuem um `@Id`** (uma Chave Primária).
2. **Value Type (Tipo de Valor):** São objetos cujo ciclo de vida depende exclusivamente da Entidade a qual pertencem. Eles **não possuem `@Id`** e não podem ser referenciados por outras entidades. 
   * *Exemplo Clássico:* As classes `String` e `Integer` do Java. 
   * *Exemplo Prático:* Em um sistema de e-commerce, a classe `Address` (Endereço) geralmente é um Value Type. Ela só existe como um "detalhe" do Cliente. Se o cliente for apagado, o endereço dele não serve para mais nada e some junto.

---

## 2. Component Mapping (Mapeamento de Componentes)
*(Aulas 22, 23 e 24)*

Quando temos uma relação de Composição envolvendo **Value Types**, utilizamos uma técnica chamada **Component Mapping**. 

Em vez de criar uma tabela inteira no banco de dados só para guardar um `Address`, nós **"embutimos" (embed)** as colunas do Endereço diretamente dentro da tabela da Entidade principal (ex: tabela `Person`). Isso otimiza o banco de dados e evita `JOINs` desnecessários.

### Como declarar no código: `@Embeddable` e `@Embedded`

**1. A Classe Componente (O Tipo de Valor):**
Usamos a anotação `@Embeddable` para avisar ao Hibernate: *"Esta classe não tem `@Id`, ela é apenas um componente que será embutido dentro de outra entidade."*
```java
@Embeddable
public class Address {
	private String street;
	private String city;
	private String zipcode;
	// Construtor vazio, getters e setters...
}
```

**2. A Classe Entidade (O Todo):**
Usamos `@Embedded` no atributo para injetar as colunas.
```java
@Entity
public class Person {
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY) 
	private Long id;
	
	@Embedded // O Hibernate pegará 'street', 'city' e 'zipcode' e criará como colunas na tabela Person!
	private Address address;	
}
```

### O Problema: Nomes de Colunas Duplicados
E se a pessoa tiver DOIS endereços (`homeAddress` e `billingAddress`)? O Hibernate tentaria criar duas colunas chamadas `street` na mesma tabela, o que gera erro no banco de dados!

**A Solução: `@AttributeOverride`**
Para resolver isso, nós "sobrescrevemos" o nome original das colunas do componente no momento em que o embutimos:
```java
@Embedded
@AttributeOverrides({
    @AttributeOverride(name="street", column=@Column(name="home_street")),
    @AttributeOverride(name="city", column=@Column(name="home_city"))
})
private Address homeAddress;

@Embedded
@AttributeOverrides({
    @AttributeOverride(name="street", column=@Column(name="billing_street")),
    @AttributeOverride(name="city", column=@Column(name="billing_city"))
})
private Address billingAddress;
```

---

## 3. Associações One-To-Many e Many-To-One
*(Aulas 25, 28 e 29)*

Aqui lidamos com relações de **Entidade para Entidade**. O exemplo clássico do curso é `Guide` (Professor/Guia) e `Student` (Aluno).
* 1 Guia orienta Muitos Alunos (**One-To-Many**).
* Muitos Alunos são orientados por 1 Guia (**Many-To-One**).

### A Regra de Ouro: O "Dono" do Relacionamento
Em um banco de dados relacional, os vínculos são feitos através de **Chaves Estrangeiras (Foreign Keys)**. 
Para o Hibernate, **a tabela que guarda fisicamente a coluna da Chave Estrangeira é sempre considerada a "Dona" (Owner) do relacionamento.**
Neste cenário, a tabela `Student` é quem vai receber a coluna `guide_id`. Portanto, `Student` é o dono!

**1. O Lado do Dono (`Student` - Lado Muitos):**
O dono define a `@JoinColumn`, que é literalmente a instrução de criar a Foreign Key no banco.
```java
@ManyToOne(cascade = {CascadeType.PERSIST})
@JoinColumn(name="guide_id") 
private Guide guide;
```

**2. O Lado Inverso (`Guide` - Lado Um):**
O lado inverso é meramente "espelho". Ele deve obrigatoriamente usar o atributo `mappedBy`. Esse atributo avisa ao Hibernate: *"Ei, eu não controlo o banco de dados. Vá olhar a variável `guide` lá na classe Student para entender o relacionamento."*
```java
@OneToMany(mappedBy="guide", cascade={CascadeType.PERSIST, CascadeType.REMOVE})
private Set<Student> students = new HashSet<Student>();	
```

### Métodos Utilitários (Importante!)
Como o lado Inverso não controla o banco, se você apenas fizer `guide.getStudents().add(aluno)`, o Hibernate **não atualizará** o banco de dados. Você deve atualizar o dono também! Para não esquecer, sempre crie um método utilitário:
```java
public void addStudent(Student student) {
    this.students.add(student); // Sincroniza o lado inverso
    student.setGuide(this);     // Sincroniza o dono! O Hibernate detecta isso e atualiza o banco.
}
```

---

## 4. Cascatas (Cascading)
*(Aulas 26 e 27)*

Na orientação a objetos, você lida com grafos profundos. Imagine salvar um `Guia`, que tem 50 `Alunos`, e cada aluno tem 2 `Endereços`. Fazer `session.persist()` 53 vezes na mão é terrível.

A cascata resolve isso. Quando configuramos `cascade={CascadeType.PERSIST}`, nós dizemos: *"Ao mandar persistir esta entidade, propague o comando para os filhos dela automaticamente"*.

---

## 5. orphanRemoval (A Remoção de Órfãos)
*(Aulas 30 e 31)*

Imagine que temos a operação de Cascata de Remoção (`CascadeType.REMOVE`).
Se você deletar um aluno (Filho), e a cascata for ativada em direção ao Guia (Pai), o Hibernate tentará apagar o Guia do banco de dados. 
Mas espere... **E se aquele Guia tiver outros alunos dependendo dele?** O banco de dados vai explodir um erro de *Constraint Violation* (Violação de Chave Estrangeira), pois você está tentando apagar um registro que deixaria outros alunos "órfãos".

**A Solução: `orphanRemoval = true`**
```java
@OneToMany(mappedBy="guide", cascade={CascadeType.PERSIST}, orphanRemoval=true)
private Set<Student> students = new HashSet<Student>();
```
O `orphanRemoval=true` é uma feature super inteligente. Ele diz: *"Se um aluno for removido da coleção `students` em memória, ou se o vínculo dele for setado para Nulo, considere-o um órfão. Vá no banco de dados e delete a linha desse aluno específico de forma segura, sem explodir violações."*

---

## 6. Mapeamento One-To-One (Um para Um)
*(Aula 32)*

Um relacionamento 1 para 1 ocorre quando uma entidade pertence exclusivamente a outra, e vice-versa. (Ex: Um `Customer` possui apenas um `Passport`, e aquele `Passport` só pertence àquele `Customer`).

**O Segredo no Banco de Dados:**
Fisicamente, um relacionamento One-To-One é criado no banco de dados exatamente igual a um Many-To-One, **porém com uma restrição `UNIQUE` (Chave Única)** na coluna da Chave Estrangeira. Isso garante que nenhum outro cliente consiga vincular o mesmo ID de passaporte.

```java
@Entity
public class Customer {
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY) 
	private Long id;
	
	@OneToOne(cascade={CascadeType.PERSIST})
	@JoinColumn(name="passport_id", unique=true) // O 'unique=true' é o coração do One-To-One
	private Passport passport;
}
```

---

## 7. Identificadores Derivados com `@MapsId`
*(Aula 33)*

Em relações One-To-One, muitas vezes a Entidade Dependente (o Customer) não precisa ter um `ID` próprio (como um Auto Increment). Ela pode simplesmente **herdar o mesmo ID** da entidade forte (o Passaporte). Se o Passaporte é `ID 5`, o Cliente ganha o `ID 5`. 

Isso elimina colunas redundantes e deixa as buscas (JOINs) mais performáticas. Fazemos isso usando a anotação `@MapsId`:

```java
@Entity
public class Customer {
	@Id
	private Long id; // ATENÇÃO: Retiramos o @GeneratedValue daqui!
	
	@OneToOne(cascade={CascadeType.PERSIST})
	@JoinColumn(name="passport_id")
	@MapsId // Mágica: "A chave primária desta classe (Customer) será copiada do objeto 'passport'"
	private Passport passport;
}
```

---

## 8. Mapeamento Many-To-Many (Muitos para Muitos)
*(Aulas 34 e 35)*

Ocorre quando Múltiplos Atores atuam em Múltiplos Filmes. Bancos Relacionais não suportam isso nativamente. Somos obrigados a criar uma **Tabela de Junção (Join Table)** no meio do caminho para mapear os IDs de ambos.

No Hibernate, usamos `@ManyToMany` e definimos como será essa Tabela de Junção usando `@JoinTable`.

**Lado do Dono (`Movie`):**
```java
@Entity
public class Movie {
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY) 
	private Long id;
	
	@ManyToMany(cascade={CascadeType.PERSIST})
    @JoinTable(
            name="movie_actor", // Nome da tabela que será gerada no banco
            joinColumns={@JoinColumn(name="movie_id")}, // Coluna que aponta para Movie
            inverseJoinColumns={@JoinColumn(name="actor_id")} // Coluna que aponta para Actor
    )	
	private Set<Actor> actors = new HashSet<Actor>();	
}
```
O lado inverso (`Actor`) usaria simplesmente `@ManyToMany(mappedBy="actors")`.

---

## 9. Mapeamento de Enums
*(Aulas 36 a 38)*

O Java permite criarmos `enums` (ex: `EmployeeStatus.FULL_TIME`). Mas como salvar isso no banco?
Por padrão, o Hibernate salva o **valor numérico ordinal** (0, 1, 2...). Isso é **extremamente perigoso**, pois se alguém no futuro mudar a ordem das opções lá no arquivo Java, os dados antigos no banco passarão a representar coisas erradas!

**Forma 1 (Simples): Salvar como Texto**
```java
@Enumerated(EnumType.STRING)
private EmployeeStatus status;
```
Isso força o Hibernate a salvar a palavra "FULL_TIME" na coluna de texto.

**Forma 2 (Profissional): AttributeConverter**
Para controle total, criamos uma classe que dita exatamente como a conversão ocorre em ambos os sentidos.
```java
public class EmployeeStatusConverter implements AttributeConverter<EmployeeStatus, Integer>{
	
	// Transforma o Enum do Java em um Integer para salvar no banco
	public Integer convertToDatabaseColumn(EmployeeStatus attr) {
        if (attr == null) return null;
        switch (attr) {
            case FULL_TIME: return 100; // Customização explícita!
            case PART_TIME: return 200;
            default: throw new IllegalArgumentException("Unknown attribute");
        }
	}
	
	// Transforma o Integer vindo do banco de volta em Enum para o Java
	public EmployeeStatus convertToEntityAttribute(Integer dbData) {
        // Lógica inversa...
	}
}
```
Na entidade, usamos: `@Convert(converter = EmployeeStatusConverter.class)`.

---

## 10. Mapeamento de Coleções de Value Types
*(Aulas 39 e 40)*

Às vezes, uma Entidade possui não apenas um valor, mas uma **Lista de Value Types** (Ex: Uma lista de `String` com apelidos, ou uma Lista de objetos `@Embeddable` Address). 

Em vez de criar uma Entidade complexa nova, usamos a anotação `@ElementCollection`. O Hibernate criará uma tabela à parte no banco para gerenciar essa lista, sem a necessidade de chaves primárias ou complexidades na classe destino.

```java
@Entity
public class Friend {
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY) 
	private Long id;

	@ElementCollection
	@CollectionTable(name = "friend_nickname", joinColumns=@JoinColumn(name = "friend_id"))
	@Column(name = "nickname")
	private Collection<String> nicknames = new ArrayList<String>();
}
```

---

## 11. Chaves Primárias Compostas (Composite Keys)
*(Aulas 41 a 43)*

### O que é uma Chave Primária Composta?
Normalmente, identificamos uma linha no banco de dados com uma única coluna `ID` (como um número de matrícula ou um ID auto-incremento). No entanto, há regras de negócio onde **uma única coluna não é suficiente para garantir a exclusividade de um registro**.

*Exemplo prático:* Uma tabela que relaciona um `Livro` a um número de `Capítulo`. Um livro pode ter vários capítulos (1, 2, 3), e outros livros também terão os capítulos 1, 2, 3. O número do capítulo isolado não é único no banco, e o ISBN do livro também não. **A exclusividade só existe quando juntamos os dois: (ISBN + Número do Capítulo).** Isso é uma Chave Composta!

### Como implementar no Java?
O Hibernate precisa comparar objetos em memória. Por isso, as colunas que formam a chave composta devem ser agrupadas em uma **Classe Específica**. E por obrigação da especificação JPA, essa classe **DEVE implementar a interface `Serializable` e DEVE sobrescrever os métodos `equals()` e `hashCode()`**, para que o Hibernate consiga entender se duas chaves são idênticas.

**A forma recomendada: `@EmbeddedId`**
Nós criamos a classe separada anotada com `@Embeddable`:
```java
@Embeddable
public class ChapterId implements java.io.Serializable {
	
	@Column(name = "ISBN", nullable = false, length = 50)
	private String isbn;
	
	@Column(name = "CHAPTER_NUM", nullable = false)
	private int chapterNum;
    
	// equals() e hashCode() gerados e obrigatórios!
}
```
E na entidade principal, usamos o `@EmbeddedId` no lugar do `@Id`:
```java
@Entity
public class Chapter {
	
	@EmbeddedId
	private ChapterId chapterId; // Esta classe contém a Chave Primária dupla!
	
	private String title;
}
```

### O Problema: Uma Chave Estrangeira que também é Chave Primária (Aula 43)

Às vezes, a modelagem do banco de dados exige que **uma das colunas da sua Chave Primária Composta seja, ao mesmo tempo, uma Chave Estrangeira (Foreign Key)** apontando para outra tabela.

*Exemplo Prático (Aula 43):* Temos uma entidade `User` (Usuário) que pertence a um `Department` (Departamento). A regra de negócio diz que a Chave Primária do Usuário é a **combinação do nome de usuário (`username`) com o ID do Departamento (`departmentId`)**. 

A classe da chave composta (`UserId`) fica assim:
```java
@Embeddable
public class UserId implements Serializable {
	@Column(name="username_cpk_col1")
    protected String username;
    
	@Column(name="derpartment_id_cpk_col2")
    protected Long departmentId; // <--- Este campo é parte da PK, mas TAMBÉM é uma Foreign Key!
	
	// construtor, equals() e hashCode()...
}
```

O grande problema é: como eu coloco a anotação `@ManyToOne` dentro de uma classe `@Embeddable`? **Você não coloca!** As anotações de relacionamento devem ficar na Entidade principal (`User`). Mas se você mapear a Foreign Key na classe `User` normalmente, o Hibernate vai se confundir, pois ele não vai saber preencher a variável `departmentId` que está presa lá dentro do `UserId`. 

### A Solução Mágica: `@MapsId`

Para unir esses dois mundos, usamos o `@MapsId`. Ele funciona como uma "ponte" entre o relacionamento e a chave composta.

```java
@Entity
@Table(name = "USERS")
public class User {

    @EmbeddedId
    private UserId userId; // Nossa chave composta
    
    @Column(nullable=false)
    private String email;

    @ManyToOne
    @JoinColumn(name="department_id_fk") // Cria a Foreign Key física no banco
    @MapsId("departmentId") // <--- A MÁGICA ACONTECE AQUI
    protected Department department;
    
    // ...
}
```

**Como o `@MapsId` funciona:**
A anotação `@MapsId("departmentId")` diz exatamente o seguinte para o Hibernate: 
> *"Eu sei que o relacionamento Many-To-One vai gerar um ID de Departamento no banco. Quando você for salvar/buscar esse ID, pegue o valor dele e **copie automaticamente** para dentro do atributo `departmentId` da minha classe `UserId`!"*

Isso resolve todo o conflito. O Hibernate agora amarra o relacionamento dinamicamente com a chave composta.

---

## 12. Book Store (Revisão Geral)
*(Aula 44)*

Nesta aula foi construído o projeto `bookstore-w-hibernate-and-jpa-annotations`, servindo como uma grande prova final do capítulo. O projeto uniu todos os conceitos de uma vez:
* Como desenhar um Grafo de Objetos Bidirecional entre `Publisher` -> `Book` -> `Chapter`.
* A utilização maciça de Cascatas (`CascadeType.PERSIST`) para salvar dezenas de tabelas de uma vez com apenas um `session.persist()`.
* A implementação complexa das Chaves Compostas com `@MapsId` abordadas na aula 43.

---

## 13. Mapeamento JSON
*(Aula 45)*

Sistemas modernos frequentemente precisam armazenar estruturas dinâmicas que não cabem bem em tabelas fixas. Os bancos de dados (como o MySQL e PostgreSQL) introduziram suporte nativo ao tipo `JSON`. 

O Hibernate 6 em diante suporta isso nativamente de forma fantástica. Basta usar a anotação **`@JdbcTypeCode(SqlTypes.JSON)`**.

```java
@Entity
public class Author {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;    
    
	@JdbcTypeCode(SqlTypes.JSON)
    private Book book; // Esta classe Book inteira virará um texto JSON em uma única coluna no Banco!
}
```
**Atenção:** Para que o Hibernate saiba como converter o objeto Java em String JSON nos bastidores, você precisa adicionar a dependência do Jackson (o formatador/serializador) no arquivo `pom.xml`, especificamente a biblioteca `jackson-module-jakarta-xmlbind-annotations`.
