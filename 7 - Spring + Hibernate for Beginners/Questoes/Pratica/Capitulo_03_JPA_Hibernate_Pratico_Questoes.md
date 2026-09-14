# Questões Práticas – Capítulo 03: JPA & Hibernate

## Exercício 3.1 – Configurar `DataSource` e `EntityManagerFactory`
**Cenário**
Crie uma classe `JpaConfig` que configure o `DataSource` (H2 em memória) e o `EntityManagerFactory` usando o `LocalContainerEntityManagerFactoryBean`.

```java
package com.example.jpa.config;

import org.springframework.context.annotation.*;
import org.springframework.jdbc.datasource.DriverManagerDataSource;
import org.springframework.orm.jpa.*;
import org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter;
import javax.sql.DataSource;
import java.util.Properties;

@Configuration
public class JpaConfig {

    @Bean // 🟢 Atividade: DataSource H2
    public DataSource dataSource() {
        DriverManagerDataSource ds = new DriverManagerDataSource();
        ds.setDriverClassName("org.h2.Driver");
        ds.setUrl("jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1");
        ds.setUsername("sa");
        ds.setPassword("");
        return ds;
    }

    @Bean // 🟢 Atividade: EntityManagerFactory
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource) {
        LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
        emf.setDataSource(dataSource);
        emf.setPackagesToScan("com.example.jpa.entity");
        emf.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
        Properties jpaProps = new Properties();
        jpaProps.put("hibernate.hbm2ddl.auto", "update");
        jpaProps.put("hibernate.dialect", "org.hibernate.dialect.H2Dialect");
        emf.setJpaProperties(jpaProps);
        return emf;
    }
}
```

---

## Exercício 3.2 – Definir entidade `Customer`
**Cenário**
Crie a entidade `Customer` com campos `id`, `firstName`, `lastName` e `email`. Use anotações JPA.

```java
package com.example.jpa.entity;

import jakarta.persistence.*;

@Entity // 🟢 Atividade: anotação JPA
@Table(name = "customers")
public class Customer {
    @Id // 🟢 Atividade: PK
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    // getters & setters
}
```

---

## Exercício 3.3 – Criar `CustomerRepository` com Spring Data JPA
**Cenário**
Implemente a interface que extende `JpaRepository<Customer, Long>` e adicione um método de busca por email.

```java
package com.example.jpa.repository;

import org.springframework.data.jpa.repository.JpaRepository;
import java.util.Optional;

public interface CustomerRepository extends JpaRepository<Customer, Long> {
    Optional<Customer> findByEmail(String email); // 🟢 Atividade: query derivada
}
```

---

## Exercício 3.4 – Serviço que usa o repositório
**Cenário**
Crie `CustomerService` que contém métodos `save`, `findById` e `findByEmail` delegando ao `CustomerRepository`.

```java
package com.example.jpa.service;

import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.Optional;

@Service
public class CustomerService {
    private final CustomerRepository repo;
    public CustomerService(CustomerRepository repo) { this.repo = repo; }

    @Transactional // 🟢 Atividade: transação
    public Customer save(Customer c) { return repo.save(c); }

    public Optional<Customer> findById(Long id) { return repo.findById(id); }

    public Optional<Customer> findByEmail(String email) { return repo.findByEmail(email); }
}
```

---

## Exercício 3.5 – Teste de integração com `@DataJpaTest`
**Cenário**
Escreva um teste que salva um `Customer` e verifica a busca por email.

```java
@DataJpaTest
class CustomerRepositoryTest {
    @Autowired private CustomerRepository repo;

    @Test
    void saveAndFindByEmail() {
        Customer c = new Customer();
        c.setFirstName("Ana"); c.setLastName("Silva"); c.setEmail("ana@example.com");
        repo.save(c);
        Optional<Customer> opt = repo.findByEmail("ana@example.com");
        Assertions.assertTrue(opt.isPresent());
    }
}
```

---

## Exercício 3.6 – Configurar `hibernate.show_sql` e `format_sql`
**Cenário**
Altere `JpaConfig` para exibir SQL no console formatado.

```java
jpaProps.put("hibernate.show_sql", "true");
jpaProps.put("hibernate.format_sql", "true"); // 🟢 Atividade: propriedades adicionais
```

---

## Exercício 3.7 – Utilizar **`@OneToMany`** e **`@ManyToOne`**
**Cenário**
Crie entidade `Order` com relação *many-to-one* para `Customer` e *one-to-many* para `OrderItem`.

```java
@Entity
public class Order {
    @Id @GeneratedValue
    private Long id;
    @ManyToOne // 🟢 Atividade: relacionamento
    private Customer customer;
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
    private List<OrderItem> items = new ArrayList<>();
    // getters/setters
}

@Entity
public class OrderItem {
    @Id @GeneratedValue
    private Long id;
    private String product;
    private int quantity;
    @ManyToOne // 🟢 Atividade: back reference
    private Order order;
    // getters/setters
}
```

---

## Exercício 3.8 – Implementar **`@BatchSize`** para otimizar consultas
**Cenário**
Aplique `@BatchSize(size = 10)` ao relacionamento `Order.items` e teste a quantidade de queries.

```java
@OneToMany(mappedBy = "order", cascade = CascadeType.ALL)
@BatchSize(size = 10) // 🟢 Atividade: otimização
private List<OrderItem> items;
```

---

## Exercício 3.9 – Configurar **Second‑level cache** com EhCache
**Cenário**
Adicione a dependência `ehcache` e configure o provider no `JpaConfig`.

```java
jpaProps.put("hibernate.cache.use_second_level_cache", "true");
jpaProps.put("hibernate.cache.region.factory_class", "org.hibernate.cache.jcache.JCacheRegionFactory");
jpaProps.put("hibernate.javax.cache.provider", "org.ehcache.jsr107.EhcacheCachingProvider"); // 🟢 Atividade: cache habilitado
```

---

## Exercício 3.10 – Persistir e carregar entidade via `EntityManager` manualmente
**Cenário**
Em um `CommandLineRunner` injete `EntityManager`, persista um `Customer` e recupere‑o usando `find`.

```java
@Component
public class JpaRunner implements CommandLineRunner {
    @PersistenceContext private EntityManager em;

    @Override
    public void run(String... args) {
        Customer c = new Customer();
        c.setFirstName("Bob"); c.setLastName("Brown"); c.setEmail("bob@example.com");
        em.persist(c); // 🟢 Atividade: persist
        em.flush();
        Customer loaded = em.find(Customer.class, c.getId()); // 🟢 Atividade: find
        System.out.println("Loaded: " + loaded.getEmail());
    }
}
```

---

**Como usar**
1. Crie o pacote `com.example.jpa...` conforme mostrado.
2. Preencha cada bloco marcado com 🟢 ou 🟡.
3. Rode a aplicação com `mvn spring-boot:run` e execute os testes.

> **Arquivo salvo em** `C:\Users\lucas\OneDrive\Documentos\Cursos\spring-boot-4-spring-7-hibernate-for-beginners\Questoes\Pratica\Capitulo_03_JPA_Hibernate_Pratico_Questoes.md`
