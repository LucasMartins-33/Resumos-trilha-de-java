# Capítulo 07: Querying

## Introdução
Neste capítulo, mergulharemos no mundo das consultas em JPA, abordando três principais tópicos:
1. Java Persistence Query Language (JPQL)
2. Comportamento de Flushing (Sincronização de Transações)
3. Criteria API

Adotaremos um tom didático, explorando os códigos-fonte reais para compreender o funcionamento interno, as configurações necessárias e as práticas recomendadas na persistência de dados.

---

## 1. Java Persistence Query Language (JPQL)

O JPQL é uma linguagem orientada a objetos para realizar consultas. Ao contrário do SQL nativo, que interage diretamente com as tabelas e colunas do banco de dados, o JPQL lida com **entidades e seus atributos**.
A grande vantagem do JPQL é a portabilidade. Como ele é independente do banco de dados, se você trocar o SGBD (de MySQL para PostgreSQL, por exemplo), suas consultas JPQL não precisarão ser alteradas, pois o provedor JPA (como o Hibernate) se encarrega de traduzi-las para o dialeto SQL correto em tempo de execução.

### Características do JPQL:
- **Case Sensitivity:** Diferente do SQL que é totalmente *case-insensitive* (insensível a maiúsculas/minúsculas), no JPQL o nome das entidades e de seus atributos são *case-sensitive*, enquanto as palavras-chave (SELECT, FROM, WHERE, etc.) são *case-insensitive*.
- **Alias:** É obrigatório (ou fortemente recomendado) atribuir um alias (variável de identificação) à entidade para navegar por seus atributos e relacionamentos (ex: `select g from Guide g`).

### 1.1. Consultas Básicas (Querying Entities)

Para buscar todos os instrutores (entidade `Guide`), criamos a consulta usando o método `createQuery` do `EntityManager`. 

```java
// Retorna a entidade Guide por completo
Query query = em.createQuery("select guide from Guide guide");
List<Guide> guides = query.getResultList();

for (Guide guide : guides) {
    System.out.println(guide);
}
```

Traduzido para SQL em tempo de execução, isso emite algo como `SELECT * FROM Guide`.

### 1.2. Seleção de Campo Único (Single Field Selection)

Se você precisa de apenas uma coluna, como o nome do instrutor, você navega pelo alias até o atributo desejado:

```java
Query query = em.createQuery("select guide.name from Guide guide");
List<String> names = query.getResultList();

for (String name : names) {
    System.out.println(name);
}
```

Isso gera no SQL: `SELECT name FROM Guide`. O resultado em Java reflete isso, trazendo uma lista de tipos `String` em vez da entidade completa.

### 1.3. Report Queries (Múltiplos Campos)

Ao gerar relatórios em que se necessita de apenas uma pequena porção de dados de muitos registros, selecionamos múltiplos atributos separados por vírgula. O retorno será uma lista de arrays de objetos (`List<Object[]>`):

```java
Query query = em.createQuery("select guide.name, guide.salary from Guide guide");
List<Object[]> resultList = query.getResultList();

for (Object[] objects : resultList) {
    // objects[0] = name, objects[1] = salary
    System.out.println("Name: " + objects[0] + ", Salary: " + objects[1]);				
}
```

### 1.4. Filtragem (Where Clause) e Wildcards

Para filtrar resultados, usamos a cláusula `WHERE`. Se for necessário buscar por partes de uma string, utilizamos a palavra-chave `LIKE` com o caractere curinga `%`.

```java
// Busca exata pelo salário
Query query = em.createQuery("select guide from Guide guide where guide.salary = 1000");

// Busca por padrão (nome do instrutor que começa com a letra 'm', case-insensitive na base SQL)
Query queryLike = em.createQuery("select guide from Guide guide where guide.name like 'm%'");
```

### 1.5. Dynamic Queries e Segurança (SQL Injection)

Construir consultas apenas concatenando strings de *inputs* de tela é uma falha gravíssima de segurança. Um usuário mal-intencionado poderia injetar comandos SQL indesejados.

**Forma ERRADA (Vulnerável):**
```java
String name = "Ian Lamb'; DROP TABLE Guide; --"; // Simulação de input malicioso
Query query = em.createQuery("select guide from Guide guide where guide.name = '" + name + "'"); // PERIGO!
```

**Forma CORRETA (Named Parameters):**
Para evitar brechas e ajudar o banco a realizar cache de compilação da query (Statements Precompilados), utilizamos parâmetros nomeados prefixados com `:` (dois pontos).

```java
Query query = em.createQuery("select guide from Guide guide where guide.name = :name");
query.setParameter("name", "Ian Lamb");
Guide guide = (Guide) query.getSingleResult(); // getSingleResult é usado quando se espera apenas 1 registro exato
```

Neste cenário, podemos também abusar do **Encadeamento de Métodos (Method Chaining)** para uma escrita fluente:
```java
Guide guide = (Guide) em.createQuery("select guide from Guide guide where guide.name = :name")
                        .setParameter("name", "Ian Lamb")
                        .getSingleResult();
```

### 1.6. Native SQL Queries

Existem ocasiões em que é extremamente necessário o uso de dialetos específicos do banco (funções proprietárias ou dicas de otimização). O JPA permite executar SQL nativo puro e até mapear seu retorno em Entidades gerenciadas, deixando o trabalho sujo por conta do provedor (Hibernate).

```java
// Ao passar 'Guide.class', o Hibernate converterá o ResultSet nativo nos objetos mapeados
Query query = em.createNativeQuery("select * from Guide", Guide.class);
List<Guide> guides = query.getResultList();
```

### 1.7. Named Queries

Para centralizar e organizar melhor o código, Named Queries permitem predefinir a consulta num arquivo XML de metadados (`orm.xml`) ou em anotações diretamente sobre as entidades, fornecendo um controle mais limpo.

*Exemplo de Mapeamento no `orm.xml`:*
```xml
<entity class="entity.Guide">
    <named-query name="findByGuide">
        <query>
            <![CDATA[ select g from Guide g where g.name = :name ]]>
        </query>
    </named-query>
</entity>
```
*Obs: É vital encapsular a query em `CDATA` para evitar conflitos dos operadores SQL (`<, >`) com o formato XML.*

*Acessando a Named Query no Cliente Java:*
```java
List<Guide> guides = em.createNamedQuery("findByGuide")
                       .setParameter("name", "Mike Lawson")
                       .getResultList();
```

### 1.8. Funções Agregadas (Aggregate Functions)

Funções como `count()`, `max()`, `min()` delegam a parte pesada do processamento para o banco de dados. Nunca traga os resultados em memória para contá-los com `.size()`.

```java
// Count sempre retornará um tipo Long
Query queryCount = em.createQuery("select count(guide) from Guide guide");
Long numOfGuides = (Long) queryCount.getSingleResult();

// O max se ajustará ao tipo mapeado da coluna solicitada
Query queryMax = em.createQuery("select max(guide.salary) from Guide guide");
Integer maximumSalary = (Integer) queryMax.getSingleResult();
```

### 1.9. Juntando e Buscando Associações (Joins & Fetching)

Mapear os relacionamentos em JPA influencia diretamente em como unimos tabelas.

**Inner Join:**
Traz apenas os estudantes que possuem uma associação ativa com um instrutor.
```java
// O "join" usa a inteligência do mapeamento @ManyToOne implicitamente para criar o ON
Query query = em.createQuery("select student from Student student join student.guide guide");
```

**Left Outer Join / Right Join:**
Recupera todos os registros da tabela da esquerda (estudantes), caso não tenham instrutor vinculado, seus campos relacionados ficarão nulos em vez de remover o estudante inteiro do resultado.
```java
Query query = em.createQuery("select student from Student student left join student.guide guide");
```

**Fetching Associations (Carregamento Eager Dinâmico):**
Relacionamentos `OneToMany` (ex: Coleção de `students` em um `guide`) adotam a estratégia `LAZY` (preguiçosa) por padrão para performance. Contudo, se a regra de negócio vai precisar dos alunos, faremos milhares de queries separadas de forma oculta pelo proxy (o problema do N+1 Selects). 

Resolvemos isso exigindo com `FETCH` que os dados atrelados entrem na primeira e única query executada.

```java
// Traz apenas o instrutor, e futuramente buscará as listas por trás dos panos (Risco N+1):
Query queryLazy = em.createQuery("select guide from Guide guide join guide.students student");

// O milagre do JOIN FETCH! Recupera de modo instantâneo, em um único pulo (ansiosamente):
Query queryFetch = em.createQuery("select guide from Guide guide join fetch guide.students student");
```

---

## 2. Comportamento de Flushing e Transações

Para dominar o JPA, é imprescindível entender o Flushing. Flushing é o mecanismo onde o Hibernate compara as cópias de trabalho no *Persistence Context* (sua memória/cache interno) através do *Dirty Checking* e descarrega SQLs de comandos DML (`UPDATE`, `INSERT`, `DELETE`) para o Banco de Dados.

Ocorre tipicamente de três maneiras:

1. **Transaction Commit:** O descarregamento final, ao invocar `em.getTransaction().commit()`. O banco sela de vez a transação.
2. **Explicito (Manual):** Ao invocar programaticamente o `em.flush()`. As mudanças fluem para a sessão de conexão sem fazer o commit oficial.
3. **Antes da Execução de uma Query (Automático Padrão!):** Este é o truque mágico do Hibernate. Antes de cada JPQL ou Native SQL, o Hibernate **verifica as modificações** na memória para que a query enxergue dados autênticos.

### Exemplo do Flush Antes da Query:
```java
Student student = em.find(Student.class, 2L);
student.setName("Sherry New3"); // Estado "Dirty": modificado apenas no Hibernate

// Nesse momento exato, antes desta JPQL executar, o Hibernate percebe a alteração e despacha o UPDATE para o Banco de Dados.
Query query = em.createQuery("select s.name from Student s where s.id = :id").setParameter("id", 2L);

// O resultado trará "Sherry New3" pois foi comitado preventivamente!
String name = (String) query.getSingleResult(); 
```

### Como Contornar (Commit Flush Mode)
Você pode otimizar as transações e proibir essa varredura pré-query. Alterando a propriedade para `FlushModeType.COMMIT`.

```java
em.setFlushMode(FlushModeType.COMMIT);
Student student = em.find(Student.class, 2L);
student.setName("Sherry Morgan"); 

// Como está em COMMIT, nenhum UPDATE acontece antes.
// A Query irá diretamente ao Banco Desatualizado!
Query query = em.createQuery("select s.name from Student s where s.id = :id").setParameter("id", 2L);
String name = (String) query.getSingleResult(); // Retorna o valor original do banco.

em.getTransaction().commit(); // Apenas no commit o novo nome "Sherry Morgan" fluirá para o BD.
```

---

## 3. Criteria API (JPA)

Mesmo com um poder semântico fantástico, a fraqueza do JPQL são os erros que se escondem em _strings_. Um erro de digitação irá estourar a aplicação **apenas no tempo de execução**.
O Criteria API é a resposta puramente em Java 100% estrito ao paradigma da tipificação, permitindo o compilador agir como segurança.

### 3.1. Estrutura Básica da Criteria

Criar uma Criteria requer instanciar e conectar blocos programáticos: o `CriteriaBuilder`, a `CriteriaQuery`, e a `Root` (cláusula FROM).

```java
CriteriaBuilder builder = em.getCriteriaBuilder();

// Define o tipo desejado de retorno na interface generificada
CriteriaQuery<Guide> criteria = builder.createQuery(Guide.class);

// A entidade Root do FROM
Root<Guide> root = criteria.from(Guide.class);
criteria.select(root); // SELECT guide FROM ...

// Prepara e executa passando para a TypedQuery, o cast se torna desnecessário!
TypedQuery<Guide> query = em.createQuery(criteria);
List<Guide> guides = query.getResultList();
```

### 3.2. Metamodelos Estáticos: Tipagem 100% Segura
Evitar referências frageis (string) exige MetaModelos (classes terminadas com *underscore*, como `Guide_`). Elas contêm propriedades estáticas, e o plugin maven `hibernate-jpamodelgen` cuida para mantê-las geradas a cada salvamento em código, anulando falhas na API.

```java
// ❌ Sem meta-modelo: Se a coluna mudar, estourará no Runtime
Path<String> namePath = root.get("name");

// ✅ Com meta-modelo: Um compilador acusaria na hora a ausência do campo "name"
Path<String> namePath = root.get(Guide_.name); 
```

### 3.3. Construções Comuns na Criteria

**Filtro Simples (Where Clause):**
```java
CriteriaBuilder builder = em.getCriteriaBuilder();
CriteriaQuery<Guide> criteria = builder.createQuery(Guide.class);
Root<Guide> root = criteria.from(Guide.class);

Path<Integer> salary = root.get(Guide_.salary); // Navegando pelo Model
criteria.where(builder.equal(salary, 1000)); // equals
criteria.select(root);
```

**Múltiplos Atributos (Report Query):**
```java
CriteriaQuery<Object[]> criteria = builder.createQuery(Object[].class);
Root<Guide> root = criteria.from(Guide.class);

Path<String> name = root.get(Guide_.name);
Path<Integer> salary = root.get(Guide_.salary);

criteria.select(builder.array(name, salary)); // SELECT guide.name, guide.salary
```

**Wildcards (LIKE):**
```java
Path<String> staffId = root.get(Guide_.staffId);
criteria.where(builder.like(staffId, "2000%")); 
```

**Contagem Agregada:**
```java
CriteriaQuery<Long> criteria = builder.createQuery(Long.class);
Root<Guide> root = criteria.from(Guide.class);

criteria.select(builder.count(root)); // Representa o COUNT()
Long numOfGuides = em.createQuery(criteria).getSingleResult();
```

### 3.4. Joins e Fetching com Criteria API

Assim como no JPQL, controlamos a profundidade usando métodos próprios da árvore da entidade:

**Inner Join Básico:**
```java
Root<Student> root = criteria.from(Student.class);

// Equivalente a: "join student.guide"
Join<Student, Guide> guide = root.join(Student_.guide);
```

**Left Outer Join Básico:**
```java
Join<Student, Guide> guide = root.join(Student_.guide, JoinType.LEFT);
```

**Fetch Join Dinâmico (Eager Loading):**
Tratamos o carregamento instantâneo da dependência a fim de aniquilar a emissão de outras queries e resolver o relacionamento Lazily:
```java
Root<Guide> root = criteria.from(Guide.class);

// Associações são resolvidas numa viagem só ao SQL.
Fetch<Guide, Student> students = root.fetch(Guide_.students, JoinType.LEFT);

// Ao juntar 1-N, registros duplicados dos Guides podem explodir no array list.
// Usamos o distinct para o framework remover na memória as duplicações de instrutores
criteria.select(root).distinct(true); 
```

### Resumo da Sessão

- Use **JPQL** pelo seu grande poder descritivo (string) para relatórios rápidos.
- Cuidado extremo em nunca concatenar entrada de dados nas strings, abuse dos **Named Parameters**.
- A propriedade do Persistence Context salva dados implicitamente em caso de alteração no meio da vida (O poder do *Flushing* no momento em que Querys são disparadas).
- Se desejar código maduro para refatorações longas, a **Criteria API com Metamodelos** provê a união do JPA com um modelo compilado incrivelmente robusto a falhas, porém verboso.
- Entenda com carinho o impacto destrutivo das conexões **LAZY**, conserte com o `JOIN FETCH` (Ponto crucial de otimização!).
