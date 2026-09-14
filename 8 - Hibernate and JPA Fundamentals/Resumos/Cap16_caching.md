# Capítulo 16: Caching (First Level, Second Level e Query Cache)

O ecossistema do Hibernate possui três camadas distintas de Caching para evitar idas desnecessárias ao banco de dados e otimizar a performance. 

---

## 1. O Cache de Primeiro Nível (L1) e a Identidade de Objetos

O Cache de Primeiro Nível (L1) é ativado por padrão e seu escopo é o **`EntityManager`**. Toda vez que uma transação fecha e o `EntityManager` é encerrado, esse cache morre.

Sua maior utilidade, além de evitar selects desnecessários, é garantir a **Identidade de Objeto**. Se você carregar o Guia de `ID=2`, o JPA alocará um espaço de memória na JVM. Se carregar o `ID=2` novamente na mesma transação, o JPA retornará a **mesma referência de memória**, evitando a loucura que seria ter dois objetos Java diferentes representando a mesma linha do banco ao mesmo tempo.

### A Pegadinha das Consultas (ID vs Query)
O cache L1 é estritamente **baseado no ID**. 
- Se você fizer `em.find(Guide.class, 2L)` duas vezes seguidas, o Hibernate dispara o `SELECT` na primeira vez, acha no L1, e na segunda vez **não faz SELECT**.
- Se você rodar uma **JPQL**, ex: `em.createQuery("select g from Guide g").getResultList()`, o Hibernate **vai emitir o SELECT** na mesma hora, porque ele não sabe, só de olhar para a query, quais IDs o banco vai devolver. Ao receber a resposta do banco com os IDs, ele checa o cache L1. Se ele achar o ID retornado no cache L1, ele descarta a String fresca que veio do banco e te devolve o objeto em cache.

---

## 2. O Cache de Segundo Nível (L2 - Shared Cache)

O L2 é um cache de nível de aplicação (pertence ao **`EntityManagerFactory`**). Se o Usuário A carregar um Guia e logo após o Usuário B quiser o mesmo Guia, o Hibernate do Usuário B pegará os dados do L2, sem ir ao banco!

### 2.1. Como Configurar (com Ehcache)
O JPA não possui uma implementação nativa pesada de L2, ele delega para bibliotecas externas (como Ehcache ou Infinispan).

No `persistence.xml`:
```xml
<!-- Liga o L2 Cache apenas nas Entidades onde você expressar isso (Melhor Prática) -->
<property name="jakarta.persistence.sharedCache.mode" value="ENABLE_SELECTIVE"/>

<!-- Provedor JCache/Ehcache 3 -->
<property name="hibernate.cache.region.factory_class" value="jcache"/>	
<property name="hibernate.jakarta.cache.provider" value="org.ehcache.jsr107.EhcacheCachingProvider"/>	
```

Nas entidades, usamos a anotação padrão do JPA:
```java
@Entity
@Cacheable // Agora o L2 sabe que deve armazenar esta classe!
public class Guide { ... }
```

### 2.2. O Problema das Coleções e Associações
Se o `Guide` está anotado com `@Cacheable` e possui uma lista de `Students`, ao buscar os Students, o Hibernate **emitirá uma query** mesmo que `Student` também seja `@Cacheable`.
**Por quê?** O cache L2 guarda dados desidratados (*dehydrated format*). O Guia guardado lá não sabe os IDs dos alunos dele! Para consertar isso, você deve ativar o cache na coleção explicitamente usando a anotação proprietária do Hibernate:

```java
// É OBRIGATÓRIO dizer qual estratégia de concorrência será usada:
@Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
@OneToMany(mappedBy="guide")
private Set<Student> students = new HashSet<Student>();
```
As estratégias variam desde `READ_ONLY` (para listas que nunca mudam, ex: Estados do País) até `READ_WRITE` (garante Consistência equivalente à regra de transação Read Committed) e `TRANSACTIONAL` (para clusters).

---

## 3. Query Cache (Cache de Resultados de Consulta)

Como vimos, queries via JPQL/SQL ignoram o L1 e L2 na hora de ir buscar os dados (porque não conhecem o ID antecipadamente). Para cachear queries que são chamadas milhares de vezes, usamos o **Query Cache**.

### 3.1. Habilitando
No `persistence.xml`:
```xml
<property name="hibernate.cache.use_query_cache" value="true"/>
```
No código Java (é mandatório avisar a query que ela será cacheada via *Hint*):
```java
Guide guide = (Guide) em.createQuery("select guide from Guide guide where guide.name = :name")
        .setParameter("name", "Ian Lamb")
        .setHint("org.hibernate.cacheable", true) // O Pulo do Gato!											  
        .getSingleResult();
```

### 3.2. Como a engrenagem funciona por trás?
O Query Cache cria dois "mapas" ocultos:
1. **StandardQueryCache:** Salva a sua Query e seus Parâmetros (A "Chave") e uma lista contendo apenas os **IDs** resultantes (O "Valor"). O Hibernate pegará esses IDs e irá varrer o **Cache L2** atrás dos objetos de verdade. (Atenção: usar Query Cache *sem* o L2 Cache habilitado nas Entidades irá disparar um *N+1 Selects* assustadoramente rápido!).
2. **UpdateTimestampsCache:** Esse é o guardião. Ele monitora a data de atualização de cada tabela do seu banco. Se a tabela `Guide` for modificada por uma inserção, update ou deleção (mesmo que numa coluna que sua query não usa), esse timestamp de alteração registrará uma data mais nova que o timestamp do seu *StandardQueryCache*. Na próxima vez que a query for chamada, o Hibernate comparará as duas datas, verá que a tabela foi mexida, marcará o cache como **Stale (Vencido)**, invalidará o cache e fará um `SELECT` fresquinho no banco.

**Conclusão e Regra de Ouro:** O Query Cache é maravilhoso, mas SÓ PODE ser usado em tabelas onde ocorrem 95% de leituras e 5% de gravações. Se for uma tabela que sofre mutação constante, o `UpdateTimestampsCache` vai ficar invalidando sua query de milissegundo em milissegundo, e o custo de gerenciar o cache será incrivelmente pior do que simplesmente ir ao banco.
