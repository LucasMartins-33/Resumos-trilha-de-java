# Capítulo 08: Identifier Generation Strategies

## Introdução
Neste capítulo, estudaremos as estratégias de geração de identificadores (IDs) na JPA e como o provedor Hibernate as implementa no banco de dados. O foco principal será o conceito de **Pre-INSERT Identifier Generation** (Geração de ID pré-inserção) e a estratégia **SEQUENCE**, incluindo as significativas mudanças de comportamento introduzidas a partir do Hibernate 6.

Adotaremos uma abordagem didática, destrinchando o processo que ocorre internamente, as chamadas SQL e como configurar a API para obter a performance e a retrocompatibilidade desejadas.

---

## 1. O que é uma Database Sequence?

Antes de falarmos sobre a estratégia em si, é importante entender o que é uma *Sequence* (Sequência) no nível do banco de dados. Uma Sequence é um recurso provido por sistemas de banco de dados relacionais (como PostgreSQL ou Oracle) que tem a única responsabilidade de gerar valores numéricos únicos e sequenciais. 

Diferente de uma coluna `AUTO_INCREMENT`, uma sequence existe independentemente de uma tabela.

**Criando uma Sequence no SQL:**
```sql
CREATE SEQUENCE minha_sequencia MINVALUE 0 START 1 INCREMENT 1;
```

**Obtendo o próximo valor (PostgreSQL):**
```sql
SELECT nextval('minha_sequencia'); -- Retorna 1, depois 2, depois 3...
```
O Hibernate é inteligente o suficiente para utilizar essas Sequences para gerar as chaves primárias de nossas entidades, de forma altamente performática.

---

## 2. Pre-INSERT Identifier Generation

Existem duas formas pelas quais o Hibernate pode lidar com o momento da atribuição de um ID para uma entidade: *Post-INSERT* e *Pre-INSERT*.

Se você usa a estratégia `GenerationType.IDENTITY` (que usa a coluna auto incremento do banco de dados), o banco só gera e conhece o ID no momento exato em que a linha é inserida (`INSERT`). Por causa disso, ao chamarmos `em.persist(student)`, o Hibernate é **obrigado a despachar o comando INSERT imediatamente** para conseguir o ID de volta e popular o objeto, quebrando assim o agrupamento (batch) de queries que geralmente só é descarregado no final da transação (`commit`). Isso é conhecido como *Post-INSERT*.

Entretanto, as estratégias **SEQUENCE** e **TABLE** agem de forma diferente. Elas são estratégias **Pre-INSERT**.

### 2.1. O Ciclo de Vida da Estratégia SEQUENCE

Quando configuramos uma entidade para utilizar `SEQUENCE`:

```java
@Entity
public class Student {
    @Id
    @GeneratedValue(strategy=GenerationType.SEQUENCE) 
    private Long id;
    
    // ...
}
```

O comportamento muda drasticamente:
1. Quando invocamos `em.persist(student)`, o Hibernate **não faz o INSERT**. Em vez disso, ele consulta a sequence no banco de dados (`SELECT nextval(...)`).
2. O banco retorna o próximo valor disponível (ex: ID `1`).
3. O Hibernate atribui esse ID ao objeto na memória (Persistence Context).
4. O `INSERT` é mantido na fila de espera e é emitido apenas no momento do `em.getTransaction().commit()`. 

Essa otimização é maravilhosa para o agrupamento (*batching*) de *statements* SQL, reduzindo o vaivém de pacotes de rede até o banco de dados e melhorando a performance geral do *commit*.

### 2.2. E quando o banco não suporta Sequences? (Ex: MySQL)

Bancos como o PostgreSQL nativamente têm um excelente suporte a Sequences e não precisam bloquear (*lock*) linhas de tabelas para gerar números rápidos. Contudo, em versões do MySQL (que originalmente não suportavam nativamente objects de sequences globais), o Hibernate tomava uma decisão arquitetural caso a estratégia SEQUENCE fosse escolhida: **Emular a sequence**.

A partir do Hibernate 5, ao usar `SEQUENCE` com o MySQL, o framework cria automaticamente uma tabela real no banco, geralmente chamada `hibernate_sequence`, contendo uma coluna `next_val`. 

O grande problema de performance nesse cenário com bancos não compatíveis nativamente é que, para emular a sequence de forma segura para múltiplas transações concorrentes, o Hibernate faz isso através de comandos com **Row-Level Locking**:

```sql
SELECT next_val FROM hibernate_sequence FOR UPDATE; -- Pega o lock da linha
UPDATE hibernate_sequence SET next_val = next_val + 1; -- Atualiza a tabela
```
Essa trava (`FOR UPDATE`) prejudica ligeiramente a simultaneidade quando várias threads tentam persistir dados ao mesmo tempo. É fundamental pesar esse conhecimento na balança ao usar MySQL em alta concorrência.

### 2.3. GenerationType.TABLE

A estratégia `TABLE` também é do tipo *Pre-INSERT*:

```java
@Id
@GeneratedValue(strategy=GenerationType.TABLE) 
private Long id;
```

Ao contrário da SEQUENCE (que tenta usar o recurso nativo do banco e só emula caso não ache), a estratégia `TABLE` **sempre** força a criação de uma tabela própria para controle de IDs (mesmo no PostgreSQL). A tabela normalmente tem duas colunas: o nome da sequence e o valor atual.
- **Vantagem:** Extrema portabilidade. A lógica funciona perfeitamente igual em absolutamente qualquer SGBD, seja Oracle, SQL Server, MySQL ou PostgreSQL.
- **Desvantagem:** Novamente, sofre pelo travamento de linha (Row-Level Locking) no banco a cada incremento do contador.

### 2.4. GenerationType.AUTO

Quando usamos `AUTO`, o JPA transfere a escolha do tipo para o *Provider* (Hibernate).
- **No antigo Hibernate 4:** Ele frequentemente optava pelo `IDENTITY` para a maioria dos dialetos.
- **Do Hibernate 5 em diante:** A estratégia padrão para o `AUTO` passou a ser o **SEQUENCE**, visando priorizar as otimizações de *Pre-INSERT* e *Batching* que mencionamos.

---

## 3. Mudanças na Estratégia SEQUENCE a partir do Hibernate 6

Foi implementada uma refatoração crucial sobre a forma como o Hibernate trabalha a estratégia `SEQUENCE` na transição do Hibernate 5 para o Hibernate 6.

### No Hibernate 5 (Sequência Única Global)
Por padrão, ao usar `GenerationType.SEQUENCE`, o Hibernate 5 compartilhava uma **única sequence genérica** (ou uma única tabela de emulação chamada `hibernate_sequence`) para gerar os IDs de **todas as entidades** da aplicação. 

Isso significava que, se você salvasse um `Guide`, ele receberia o ID 1. Se em seguida salvasse um `Student`, ele ganharia o ID 2. Se voltasse a salvar um `Guide`, ganharia o ID 3. Os IDs cresciam compartilhando a mesma fonte, sem colisão, mas ficando espaçados nas suas próprias tabelas.

### No Hibernate 6 (Sequências Independentes)
No Hibernate 6, a convenção padrão foi alterada para criar uma **sequence separada por entidade**.
O banco de dados passa a possuir objetos dedicados: `guide_SEQ` para a tabela de instrutores, e `student_SEQ` para a tabela de estudantes. Assim, a contagem é mantida isolada para cada classe de domínio, e os IDs das tabelas começam em 1 e progridem individualmente, gerando resultados mais limpos.

### Retrocompatibilidade: Como forçar o comportamento antigo?

Se você está migrando um projeto em produção do Hibernate 5 para o Hibernate 6, essa mudança de comportamento dos IDs é desastrosa, porque tentaria reiniciar contadores a partir de novas sequences dedicadas.

Para obrigar o Hibernate 6 a retornar para o modo antigo, ou seja, continuar utilizando **apenas uma sequence central** (`hibernate_sequence`) para todas as classes da aplicação, você deve alterar as propriedades de conexão no `persistence.xml`, definindo a estratégia de nomeação estrutural do banco como `single`:

```xml
<properties>
    <!-- Outras configs do banco omitidas -->

    <!-- As 3 opções disponíveis no Hibernate 6 são "single", "legacy" e "standard" -->
    <!-- "standard" é a padrão e cria sequences como "student_seq" -->
    <!-- "single" força o uso compartilhado da "hibernate_sequence", exatamente como no Hibernate 5 -->
    <property name="hibernate.id.db_structure_naming_strategy" value="single" />
</properties>
```

Configurando `hibernate.id.db_structure_naming_strategy` com o valor `single`, você retém 100% da identidade da arquitetura herdada do Hibernate 5. 

### Resumo do Fluxo do Exemplo (Single vs Standard)
Imagine o código:
```java
Guide guide = new Guide("2000MO10789", "Mike Lawson", 1000);
Student student = new Student("2014AL50456", "Amy Gill");

em.persist(guide);
em.persist(student);
```
- **Se `strategy = standard` (Hib 6 padrão):** `Guide` ganha ID=1 (via `guide_SEQ`) e `Student` ganha ID=1 (via `student_SEQ`).
- **Se `strategy = single` (Padrão Antigo):** `Guide` ganha ID=1 (via `hibernate_sequence`), e em seguida a sequence avança. O `Student` ganhará ID=2. Ambos não colidem globalmente.
