# Capítulo 13: N+1 SELECTs Problem, Batch Fetching and Entity Graphs

## Introdução
Neste capítulo, abordaremos um dos problemas de performance mais infames no mundo do mapeamento objeto-relacional (ORM): o **Problema do N+1 SELECTs**. Entender sua causa raiz e as três principais formas de combatê-lo (*JOIN FETCH*, *Batch Fetching* e *Entity Graphs*) é obrigatório para qualquer desenvolvedor que deseje construir aplicações JPA escaláveis e performáticas.

---

## 1. O Problema do N+1 SELECTs

### 1.1. O que é e por que acontece?
O problema do N+1 SELECTs ocorre quando executamos **1** consulta para buscar uma lista de registros-pai (ex: N Estudantes) e, em seguida, o JPA executa **N** consultas adicionais (uma para cada estudante) para buscar seus respectivos registros-filhos (ex: O Guia de cada estudante).
No fim das contas, para carregar os dados de N estudantes, o banco recebe **N + 1** comandos `SELECT`. Se você tiver 10.000 estudantes, a aplicação executará 10.001 consultas. Isso aniquila a rede e derruba o banco de dados.

**Causa Raiz:** Associações de ponto único (`@ManyToOne` e `@OneToOne`) são, por padrão, resolvidas com a estratégia `EAGER` (Ansiosa). Isso significa que se você apenas der um `SELECT s FROM Student s`, o Hibernate emitirá consultas adicionais automáticas para trazer todos os Guias da base associados àqueles estudantes.

### 1.2. Como resolver o problema
A correção deste pesadelo envolve duas etapas fundamentais:

**Passo 1: Alterar a estratégia de Fetch para LAZY**
No mapeamento de `Student`, force o relacionamento a ser preguiçoso. Isso dirá ao Hibernate para injetar um *Proxy* do Guia em vez de disparar uma consulta.
```java
@ManyToOne(fetch = FetchType.LAZY) // Fundamental!
@JoinColumn(name="guide_id")
private Guide guide;
```

**Passo 2: E quando eu realmente precisar do Guia associado?**
Mesmo com `LAZY`, se você iterar sobre a lista de Estudantes e invocar `student.getGuide().getName()`, o Proxy será ativado e o N+1 SELECTs retornará!
Para resolver isso de forma inteligente e carregar os dados de forma *Ansiosa (Eager)* apenas para aquela necessidade pontual, usamos a cláusula **`LEFT JOIN FETCH`** em nossa JPQL:

```java
// O JOIN FETCH obriga o Hibernate a trazer Estudantes e Guias em UM ÚNICO comando SQL.
Query query = em.createQuery("SELECT student FROM Student student LEFT JOIN FETCH student.guide");
List<Student> students = query.getResultList();	
```

---

## 2. Batch Fetching (O Meio-Termo)

E se tivermos **10.000** estudantes e quisermos evitar o gigantesco impacto de memória ao trazer todos com `JOIN FETCH`, mas também quisermos fugir da lentidão de 10.000 consultas ao usar `LAZY`? 
A solução para esse dilema é o **Batch Fetching** (Busca em Lotes), uma estratégia que **otimiza o Lazy Loading**.

**Como funciona?**
Nós anotamos a classe que será buscada sob demanda (ex: `Guide`) com a anotação proprietária do Hibernate `@BatchSize`.

```java
import org.hibernate.annotations.BatchSize;

@Entity
@BatchSize(size = 4) // Agrupa as consultas em lotes de 4
public class Guide { ... }
```
**O efeito prático:**
O carregamento de `Student` continua `LAZY`. Mas quando o loop invocar `student.getGuide().getName()` no primeiro estudante, o Hibernate não fará um `SELECT` apenas para aquele Guia. Ele antecipará a demanda e buscará **4 guias de uma só vez** (`SELECT ... WHERE id IN (?, ?, ?, ?)`).
Isso reduz o número astronômico de 10.000 consultas adicionais para apenas **2.500 consultas**. Você controla o impacto na rede manipulando o valor do `size`!

---

## 3. Entity Graphs (Grafos de Entidade)

Os **Entity Graphs** servem para resolver a mesma vertente: **Sobrescrever a estratégia de LAZY para EAGER dinamicamente em tempo de execução**, sem precisarmos engessar nossa entidade original e sem dependermos estritamente do `JOIN FETCH` escrito via string JPQL.

### 3.1. Definindo o Grafo na Entidade
No topo da nossa classe `Student`, declaramos o "Nó" que gostaríamos de carregar ativamente.
```java
@Entity
@NamedEntityGraph(
    name = "Guide.students", // Nome amigável do Grafo
    attributeNodes = {
        @NamedAttributeNode("students") // Nome do atributo que queremos EAGER
    }
)
public class Guide { ... }
```

### 3.2. Acionando o Grafo no EntityManager
Na hora de efetuar a consulta, passamos o Grafo recém-criado através de uma **Hint** (`Dica`) para a Query. O Hibernate fundirá as informações e carregará o estudante e seu guia numa tacada só.

```java
// Recuperando o grafo declarado
EntityGraph<?> graph = em.getEntityGraph("Guide.students");

// Usando o EntityGraph numa consulta JPQL
Query query = em.createQuery("SELECT student FROM Student student")
                .setHint("jakarta.persistence.loadgraph", graph); // Injeção da Hint

List<Student> students = query.getResultList();	
```
*Dica:* Isso funciona brilhantemente com o `em.find` padrão também!
```java
Map<String,Object> props = new HashMap<String Object>();
props.put("jakarta.persistence.loadgraph", em.getEntityGraph("Guide.students"));
Guide guide = em.find(Guide.class, 2L, props);
```

Com o domínio dessas 3 armas (*Left Join Fetch*, *Batch Fetching* e *Entity Graphs*), você está totalmente equipado para lidar com e destruir o famigerado N+1 Selects Problem em qualquer projeto real.
