# Capítulo 6: Estratégias de Busca (Fetching), Aprimoramento de Bytecode e Igualdade

Olá, aluno(a)! Neste capítulo 6, focaremos pesadamente no **desempenho da nossa aplicação** e em boas práticas essenciais do Java. Trabalharemos as Estratégias de Busca (Fetching Strategies), aprenderemos como usar o Aprimoramento de Bytecode (Bytecode Enhancement) para otimizar os nossos atributos, e finalmente entenderemos os perigos invisíveis de ignorarmos o `equals()` e `hashCode()` no mundo do ORM.

Pegue seu bloco de notas e acompanhe cada detalhe, pois o que veremos aqui separa os sistemas amadores das aplicações corporativas de alto desempenho!

---

## 1. Revisão Rápida: SQL Joins (Opcional)

Antes de falarmos de estratégias de busca da JPA, é essencial entender o comportamento de junção dos bancos relacionais, já que a JPA as utiliza por baixo dos panos.
* **INNER JOIN (ou apenas JOIN):** Retorna APENAS as linhas que possuem correspondência em ambas as tabelas (Tabela A e Tabela B). Se um lado for nulo, a linha inteira não é retornada.
* **LEFT OUTER JOIN (ou apenas LEFT JOIN):** Retorna TODAS as linhas da tabela da esquerda (Tabela A). Se houver correspondência na tabela da direita (Tabela B), os dados dela vêm junto. Se não houver, as colunas da direita virão preenchidas com `NULL`.

---

## 2. Lazy Fetching (Busca Preguiçosa) e Proxies

O conceito mais vital de performance em JPA chama-se **Lazy Fetching** (Busca Preguiçosa). Para ilustrar, se carregarmos um guia (`Guide`) que possui milhares de alunos (`Student`) na relação `@OneToMany`, nós realmente queremos carregar esses milhares de alunos imediatamente num `SELECT` pesado? Quase sempre a resposta é não.

A JPA lida com isso entregando um **Proxy**! Um Proxy é um objeto falso (um espaço reservado/placeholder). Ele aparenta ser a coleção de alunos, mas está vazio e ainda não disparou nenhum SQL contra o banco de dados.

### O Padrão de Comportamento (Default Strategies)

1. **Associações Colecionáveis (`@OneToMany`, `@ManyToMany`):** 
   O comportamento padrão é **LAZY (Preguiçoso)**. O Hibernate NÃO carrega os filhos no momento em que a entidade pai é carregada.
2. **Associações de Ponto Único (`@ManyToOne`, `@OneToOne`):**
   O comportamento padrão é **EAGER (Ansioso)**. O Hibernate sempre fará um Join imediatamente para carregar o objeto associado.

**Exemplo prático de Inicialização de Coleção LAZY:**
```java
// O Guide é carregado, mas a lista de Students vem apenas como um PROXY vazio
Guide guide = em.find(Guide.class, 2L); 

// Ainda NÃO emitiu o SELECT pros estudantes! Apenas pegou a referência local do Proxy.
Set<Student> students = guide.getStudents(); 

// AQUI SIM! Chamar um método da coleção (como .size() ou iterar num laço FOR)
// força o Proxy a se inicializar, obrigando a JPA a emitir o SELECT dos alunos.
int numberOfStudents = students.size(); 
```

> [!WARNING]
> **A Temida `LazyInitializationException`:**
> O Proxy de uma associação LAZY só tem a permissão de ir ao Banco de Dados buscar dados se a sua transação/EntityManager **ainda estiver aberta**. Se você fizer `em.close()` e DEPOIS tentar dar um `students.size()`, o Hibernate lançará a terrível exceção `LazyInitializationException`. Lembre-se: O Lazy Fetching nunca funciona fora do escopo de um `EntityManager` ativo!

### Mudando EAGER para LAZY (Recomendado)

O uso de EAGER é geralmente uma prática perigosa, pois carrega lixo na memória. O ideal é mudar as anotações `@ManyToOne` e `@OneToOne` para LAZY:
```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name="guide_id")
private Guide guide;
```
Ao fazer isso, carregar um `Student` não trará mais os dados de `Guide` imediatamente por meio de Left Outer Joins pesados!

---

## 3. O Método `getReference` e Convertendo Dados Removidos (Transient)

E se nós quiséssemos apenas associar um `Student` a um `Guide`, sabendo que o `Guide` possui o ID 3, mas **sem executar nenhum SELECT** no banco?

É para isso que usamos o método **`em.getReference()`** no lugar do `em.find()`. O `getReference` constrói diretamente um Proxy falso com o ID 3 sem perguntar ao banco se ele existe. Ele economiza consultas:

```java
// NÃO EMITE SELECT!
Student student = em.getReference(Student.class, 3L); 

// Acessar um atributo real dispara o SELECT (Inicialização do proxy)
String name = student.getName(); 
// Alternativamente, podemos forçar a inicialização usando: Hibernate.initialize(student);
```

### Lidando com Dados Apagados (Use Identifier Rollback)

Quando deletamos uma entidade através de `em.remove(objeto)`, ela entra no estado Removido (Removed). Todavia, ela continua mantendo o seu identificador original (o seu ID). Se decidirmos salvá-la novamente (ex: "Desfazer a exclusão do usuário"), chamar `em.persist()` nesse estado pode ser problemático se quisermos um **NOVO ID**.

A solução é usar a propriedade `hibernate.use_identifier_rollback=true` no nosso `persistence.xml`:
```xml
<!-- Reseta o identificador de volta para "null" após a remoção de uma entidade -->				
<property name="hibernate.use_identifier_rollback" value="true" />
```
Ao ativar essa função, se chamarmos `em.remove(student)`, o atributo `id` passará a ser `null`, transformando instantaneamente o nosso objeto que estava em estado "Removed" num estado "Transiente" regular. Se fizermos o persist novamente através de um novo `EntityManager`, ele receberá um novo e fresco ID auto-incrementado.

---

## 4. Ordenando Listas com `@OrderBy`

Para forçar coleções associadas a virem já ordenadas do banco de dados (o que gera a cláusula `ORDER BY` no SQL final do Hibernate), utilizamos a anotação `@OrderBy` da JPA.
A ordem informada nesta anotação afeta estritamente a fase de recuperação (SELECT) e não a persistência. A anotação funciona inclusive na interface `Set`.

```java
@OneToMany(mappedBy="guide", cascade={CascadeType.PERSIST})
@OrderBy("name DESC") // Ordenar estudantes pelo atributo 'name' de forma Descendente (Z a A)
// @OrderBy("name DESC, enrollmentId ASC") -> Exemplo com duas colunas
private Set<Student> students = new HashSet<Student>();
```
*Detalhe:* Se deixarmos a anotação pura `@OrderBy()` sem parâmetros, a lista será ordenada por padrão pela coluna de chave primária (`ID`) de forma Ascendente.

---

## 5. Aprimoramento de Bytecode (Bytecode Enhancement)

Nós aprendemos a fazer o *Lazy Fetching* de relacionamentos/coleções, mas e os atributos comuns da própria tabela? Por padrão, a JPA busca (Eager Fetching) **todas** as colunas primitivas. E se tivéssemos um campo de texto colossal (uma postagem de blog ou arquivo codificado de 32 mil caracteres)? Puxar isso à toa detonaria nossa rede e RAM.

Para colocar um simples atributo `String` no modo LAZY, a JPA especifica o `@Basic(fetch=FetchType.LAZY)`. No entanto, **o provedor JPA não tem obrigação de implementar isso**. O Hibernate exige que utilizemos o "Bytecode Enhancement" no empacotamento Maven para poder interceptar a criação da classe.

### Passos para criar atributos escalares preguiçosos:

**Passo 1:** Configurar a anotação na entidade. Podemos usar a anotação da JPA e a do Hibernate para dividir em "Grupos".
```java
// O @JdbcTypeCode(SqlTypes.LONGVARCHAR) mapeia no BD para colunas gigantescas
@JdbcTypeCode(SqlTypes.LONGVARCHAR) 
@Basic(fetch = FetchType.LAZY)
@Column(name = "text_content")
private String textContent;

@LazyGroup("foo") // Coloca o campo noutro grupo, para nÃ£o carregar tudo junto
@Basic(fetch = FetchType.LAZY)
@Column(name = "posted_on")
private LocalDate postedOn;
```
*(Se não usarmos `@LazyGroup`, todos os atributos marcados como `LAZY` caem num grupo "Default". Assim que o programador der o 'get' num campo bobo como `postedOn`, o Hibernate puxará inadvertidamente também o pesado `textContent`. Separar por `@LazyGroup` permite que disparem SELECTs independentes!)*

**Passo 2:** Modificar o `pom.xml` para embutir o Plugin do Hibernate no compilador:
```xml
<plugin>
    <groupId>org.hibernate.orm.tooling</groupId>
    <artifactId>hibernate-enhance-maven-plugin</artifactId>
    <version>6.2.0.Final</version>
    <executions>
        <execution>
            <configuration>
                <enableLazyInitialization>true</enableLazyInitialization>
            </configuration>
            <goals><goal>enhance</goal></goals>
        </execution>
    </executions>
</plugin>
```

---

## 6. O Perigo de não implementar `Equals` e `HashCode`

No Java, ao inserir objetos em coleções que não aceitam duplicações (como a interface `Set`), a linguagem utiliza os métodos `hashCode()` e `equals()`. O grande problema é que a herança de `Object` compara as instâncias usando o **endereço físico da memória**.
Duas consultas independentes ao mesmo ID no banco, usando dois `EntityManager` separados, gerarão **duas instâncias físicas distintas** no Java. Consequentemente, o código considerará como sendo dois estudantes diferentes e vai estragar todas as verificações como o `.contains()`.

A regra de ouro da persistência: **Sempre sobreecreva o `equals()` e o `hashCode()` em suas Entidades.** Ao fazer isso, não baseie-se em identificadores nulos, baseie-se nas **chaves de negócio (Business Keys)** naturais (ex: Cpf, Matrícula, Email, etc.).

A Apache fornece as excelentes classes `EqualsBuilder` e `HashCodeBuilder` da biblioteca Commons-Lang para simplificar isso:

```java
import org.apache.commons.lang3.builder.EqualsBuilder;
import org.apache.commons.lang3.builder.HashCodeBuilder;

@Entity
public class Student {
	
    // ID do BD auto-incrementado (pode ser nulo transitoriamente, não use para Equals!)
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY)
    private Long id;
    
    // Nossa Business Key obrigatória e única do mundo real (Matrícula)
    @Column(name="enrollment_id", nullable=false)
    private String enrollmentId;	

    @Override
    public int hashCode() {
        return new HashCodeBuilder().append(enrollmentId).toHashCode();
    }
    
    @Override
    public boolean equals(Object obj) {
        if (!(obj instanceof Student)) return false;
        Student other = (Student) obj;
        return new EqualsBuilder().append(enrollmentId, other.enrollmentId).isEquals();
    }
}
```

> [!NOTE]
> **A "Armadilha" do Cache de Primeiro Nível (Guaranteed Object Identity):**
> Se você carregar o Estudante 2, e logo em seguida o Guia que carrega o mesmo Estudante 2, sob o **mesmo `EntityManager`**, a JPA usará o cache. A referência de memória apontará exatamente para o mesmo objeto no Java. Num cenário de mesmo EntityManager, o `Set` não precisaria do seu `equals()` customizado, pois a própria JVM notaria que os dois apontam pra exata mesma alocação. Porém, aplicações corporativas sempre espalham a atividade em sessões e transações assíncronas. Portanto, é imprescindível e vital criar o contrato de igualdade de negócios!

### Considerações Finais
Neste capítulo desvendamos o controle total de inicializações (Lazy/Eager), melhoramos atributos pesados usando Bytecode Enhancement e fixamos um pilar de integridade (Equals e HashCode). Mantenha essas otimizações em mente quando for mapear os projetos!
