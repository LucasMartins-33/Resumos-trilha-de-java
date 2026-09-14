# Questões Práticas – Capítulo 09: JPA/Hibernate Avançado

## Exercício 9.1 – Configurar Second‑Level Cache com EhCache
**Cenário**
Adicione a dependência `ehcache` e configure o cache de segundo nível no `JpaConfig`.

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
    <version>3.10.8</version>
</dependency>
```

```java
// Dentro de JpaConfig
jpaProps.put("hibernate.cache.use_second_level_cache", "true");
jpaProps.put("hibernate.cache.region.factory_class", "org.hibernate.cache.jcache.JCacheRegionFactory");
jpaProps.put("hibernate.javax.cache.provider", "org.ehcache.jsr107.EhcacheCachingProvider");
```

---

## Exercício 9.2 – Definir `@Cacheable` em repositório
**Cenário**
Cacheie a consulta `findByEmail` do `CustomerRepository`.

```java
@Repository
public interface CustomerRepository extends JpaRepository<Customer, Long> {
    @Cacheable("customersByEmail") // 🟢 Atividade: habilitar cache
    Optional<Customer> findByEmail(String email);
}
```

---

## Exercício 9.3 – Utilizar `Entity Graph` para fetch eager controlado
**Cenário**
Crie um `@EntityGraph` que carregue a coleção `orders` de `Customer` em uma única query.

```java
@Entity
@NamedEntityGraph(name = "customer-with-orders", attributeNodes = @NamedAttributeNode("orders"))
public class Customer { /* ... */ }
```

```java
public interface CustomerRepository extends JpaRepository<Customer, Long> {
    @EntityGraph(value = "customer-with-orders", type = EntityGraph.EntityGraphType.LOAD)
    Optional<Customer> findWithOrdersById(Long id); // 🟢 Atividade: usar graph
}
```

---

## Exercício 9.4 – Aplicar `@BatchSize` em relacionamento `@OneToMany`
**Cenário**
Ajuste o carregamento da coleção `orderItems` em `Order` para batches de 10.

```java
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
@BatchSize(size = 10) // 🟢 Atividade: otimização batch
private List<OrderItem> orderItems = new ArrayList<>();
```

---

## Exercício 9.5 – Configurar `hibernate.show_sql` e `format_sql`
**Cenário**
Altere o `JpaConfig` para exibir SQL formatado no console.

```java
jpaProps.put("hibernate.show_sql", "true");
jpaProps.put("hibernate.format_sql", "true"); // 🟢 Atividade: propriedades adicionais
```

---

## Exercício 9.6 – Implementar **Optimistic Locking**
**Cenário**
Adicione a coluna `@Version` na entidade `Product` e teste concorrência.

```java
@Entity
public class Product {
    @Id @GeneratedValue
    private Long id;
    private String name;
    private BigDecimal price;
    @Version // 🟢 Atividade: controle de versão
    private Integer version;
    // getters/setters
}
```

```java
// Teste concorrente
@Test
void optimisticLock() {
    Product p1 = repo.findById(1L).get();
    Product p2 = repo.findById(1L).get();
    p1.setPrice(BigDecimal.TEN);
    repo.save(p1);
    // p2 ainda tem versão antiga → ao salvar gera OptimisticLockException
    Assertions.assertThrows(ObjectOptimisticLockingFailureException.class, () -> repo.save(p2));
}
```

---

## Exercício 9.7 – Implementar **Soft Delete** com `@Where` e `@SQLDelete`
**Cenário**
Marque a entidade `Customer` como `deleted = true` ao excluir, e filtre registros não deletados.

```java
@Entity
@SQLDelete(sql = "UPDATE customers SET deleted = true WHERE id=?") // 🟢 Atividade: soft delete
@Where(clause = "deleted = false") // 🟢 Atividade: filtro padrão
public class Customer {
    @Id @GeneratedValue
    private Long id;
    private String name;
    private boolean deleted = false;
    // ...
}
```

---

## Exercício 9.8 – Configurar **Stateless Session** para bulk operations
**Cenário**
Use `entityManager.unwrap(Session.class).setDefaultReadOnly(true)` para inserções massivas.

```java
@Autowired private EntityManager em;

public void bulkInsert(List<Product> products) {
    Session session = em.unwrap(Session.class);
    session.setDefaultReadOnly(true); // 🟢 Atividade: sessão read‑only
    for (Product p : products) {
        em.persist(p);
    }
    em.flush();
    session.setDefaultReadOnly(false);
}
```

---

## Exercício 9.9 – Utilizar **JPA Criteria API** para consultas dinâmicas
**Cenário**
Construa uma busca por `price` entre valores mínimo e máximo usando Criteria.

```java
public List<Product> findByPriceRange(BigDecimal min, BigDecimal max) {
    CriteriaBuilder cb = em.getCriteriaBuilder();
    CriteriaQuery<Product> cq = cb.createQuery(Product.class);
    Root<Product> root = cq.from(Product.class);
    Predicate between = cb.between(root.get("price"), min, max);
    cq.select(root).where(between);
    return em.createQuery(cq).getResultList(); // 🟢 Atividade: Consulta dinâmica
}
```

---

## Exercício 9.10 – Testar transação com `@Transactional` e rollback
**Cenário**
Crie um serviço que salva dois `Order` dentro de uma mesma transação; force exceção para validar rollback.

```java
@Service
public class OrderService {
    @Transactional // 🟢 Atividade: gerenciar transação
    public void createTwoOrders(Order o1, Order o2) {
        repo.save(o1);
        repo.save(o2);
        if (o2.getAmount() > 1000) {
            throw new IllegalArgumentException("Valor muito alto"); // força rollback
        }
    }
}
```

```java
@Test
void rollbackOnError() {
    Order o1 = new Order(); o1.setAmount(100);
    Order o2 = new Order(); o2.setAmount(2000);
    Assertions.assertThrows(IllegalArgumentException.class, () -> service.createTwoOrders(o1, o2));
    Assertions.assertTrue(repo.findAll().isEmpty()); // 🟢 Verifica rollback
}
```

---

**Como usar**
1. Crie os pacotes `com.example.advanced.jpa` (entities, repos, services, configs).
2. Preencha cada bloco marcado com 🟢/🟡.
3. Rode a aplicação e execute os testes (`mvn test`).

> **Arquivo salvo em** `C:\Users\lucas\OneDrive\Documentos\Cursos\spring-boot-4-spring-7-hibernate-for-beginners\Questoes\Pratica\Capitulo_09_JPA_Hibernate_Advanced_Pratico_Questoes.md`
