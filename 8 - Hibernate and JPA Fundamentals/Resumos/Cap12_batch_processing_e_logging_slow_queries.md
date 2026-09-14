# Capítulo 12: Batch Processing e Logging Slow Queries

## Introdução
À medida que as aplicações crescem, a escalabilidade e a performance tornam-se fatores cruciais. Processar milhares de registros um a um ou perder o controle sobre a lentidão das consultas são receitas certas para o desastre em produção. Neste capítulo, abordamos duas técnicas fundamentais: **Batch Processing** (Processamento em Lote) e **Logging de Consultas Lentas** (Slow Queries).

---

## 1. Batch Processing (Processamento em Lote)

### 1.1. O Problema da Persistência em Massa
Imagine que você precisa inserir 100.000 livros de uma só vez no banco de dados dentro de uma mesma transação. Se você criar um loop executando `em.persist(book)` repetidamente, enfrentará graves problemas:
1. **Out of Memory (Estouro de Memória):** Todas as 100.000 instâncias de `Book` ficarão presas no Contexto de Persistência (*Persistence Context*). A memória RAM do Java se esgotará rapidamente.
2. **Transaction Timeout:** Processar tanto dado de uma vez fará com que a transação estoure o tempo limite antes do `commit`.
3. **Network Round-trips (Viagens na Rede):** Ao dar o `commit`, o JPA enviará **100.000 comandos de `INSERT` individuais** para o banco. São 100.000 viagens de ida e volta pela rede, o que destruirá a performance do sistema.

### 1.2. A Solução: Processamento em Lotes
Para evitar isso, dividimos o processamento em pequenos pacotes (batches). 
O processo exige configuração em duas frentes: no arquivo de configuração do JPA e no próprio código Java.

**Passo 1: Habilitar o Batching no JDBC (`persistence.xml`)**
Você deve dizer ao Hibernate para agrupar os comandos SQL (INSERT, UPDATE, DELETE) em pacotes antes de dispará-los na rede.
```xml
<!-- Define que os comandos SQL serão enviados em grupos de 5 -->
<property name="hibernate.jdbc.batch_size" value="5" />
```

**Passo 2: Controlar a Memória no Java**
No código cliente, temos que invocar ativamente a sincronização com o banco e depois limpar a memória:
```java
em.getTransaction().begin();		

for (int i = 1; i <= 25; i++) {
    Book book = new Book("Title-" + i, "ISBN-" + i);
    em.persist(book);
    
    // Quando atingirmos o tamanho do lote (5)
    if (i % 5 == 0) {
        em.flush(); // 1. Força o envio do lote de 5 comandos INSERT para o banco.
        em.clear(); // 2. Esvazia o Contexto de Persistência, liberando memória e deixando os objetos prontos para o Garbage Collector!
    }
}
em.getTransaction().commit();
```

### 1.3. A Armadilha da Estratégia IDENTITY
Existe uma regra crucial: **O Batch Processing de INSERTs NÃO FUNCIONA com a estratégia de geração de ID `GenerationType.IDENTITY`!**

**Por quê?**
Porque a estratégia `IDENTITY` exige que o banco auto-incremente o ID e o devolva. Para que o Hibernate conheça o ID de um objeto recém persistido (já que ele precisa rastreá-lo internamente), ele é **forçado a enviar o comando `INSERT` imediatamente** no exato milissegundo em que `em.persist()` é invocado. 
Sendo assim, o Hibernate ignora solenemente a propriedade `hibernate.jdbc.batch_size` e envia comandos individuais sem agrupá-los em lotes.

---

## 2. Rastreando Consultas Lentas (Logging Slow Queries)

Em ambientes de produção, é comum que algumas consultas (queries) passem a demorar mais tempo conforme o volume de dados cresce. A partir do **Hibernate 5.4**, podemos configurar alertas automáticos para avisar quando uma query for mais lenta que um limite de tempo estabelecido.

São apenas dois passos de configuração:

**Passo 1: Definir o Tempo Limite (`persistence.xml`)**
Definimos um limite tolerável em milissegundos. Qualquer query que demorar mais que isso será tratada como lenta.
```xml
<!-- Marca como lenta qualquer consulta que demorar mais de 1 milissegundo -->
<property name="hibernate.session.events.log.LOG_QUERIES_SLOWER_THAN_MS" value="1" />
```

**Passo 2: Habilitar o Log de Queries Lentas (`log4j.properties`)**
Devemos ajustar o log4j para mostrar especificamente a categoria de queries lentas:
```properties
log4j.logger.org.hibernate.SQL_SLOW=INFO
```
Ao executar a aplicação, o log do console emitirá alertas explícitos mostrando o tempo gasto e a sintaxe das consultas problemáticas.

---

## 3. Otimizando Consultas Lentas: Criação de Índices (`@Index`)

Uma das formas mais diretas de resolver problemas de lentidão detectados pelo *Slow Query Log* é através da criação de Índices no Banco de Dados. Índices aceleram drasticamente leituras, agrupamentos e ordenações (`ORDER BY`).

Podemos comandar o JPA para criar esses índices diretamente via anotação `@Table`:

```java
@Entity
@Table(indexes = {
    // Cria um índice que engloba as colunas 'title' (em ordem ascendente) e 'isbn'
    @Index(columnList = "title asc, isbn") 
})
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;    
    private String isbn;
    
    //...
}
```
Isso fará com que o Hibernate emita comandos `CREATE INDEX` no momento em que gera/atualiza as tabelas no banco de dados, aliviando o gargalo de leituras futuras.
