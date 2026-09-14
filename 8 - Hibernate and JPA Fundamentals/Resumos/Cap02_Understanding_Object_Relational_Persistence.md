# Capítulo 02: Understanding Object/Relational Persistence

Neste capítulo, abordamos os conceitos fundamentais sobre persistência de dados utilizando Java e Bancos de Dados Relacionais, focando nos problemas que surgem ao tentar unir esses dois mundos e como o ORM (Object Relational Mapping) resolve essas questões.

---

## Aula 2: Object Relational Impedance Mismatch
O **Object Relational Impedance Mismatch** (Descompasso de Impedância Objeto-Relacional) ocorre porque linguagens Orientadas a Objetos (como Java) representam dados através de grafos de objetos interconectados, enquanto os Bancos de Dados Relacionais (RDBMS) representam os dados em formato de tabelas. 

Quando tentamos carregar ou salvar esses grafos de objetos em um banco relacional, nos deparamos com **5 problemas principais (Mismatches)**:

1. **Granularity Mismatch (Descompasso de Granularidade):**
   * **Problema:** Em Java, objetos podem ter vários níveis de granularidade (ex: uma classe `Person` contém um objeto da classe `Address`). No modelo relacional, a granularidade é limitada a apenas dois níveis: tabelas e colunas.
   * **Efeito:** Frequentemente, o modelo de objetos terá mais classes do que o número correspondente de tabelas no banco de dados.

2. **Subtype / Inheritance Mismatch (Descompasso de Herança/Subtipos):**
   * **Problema:** Java possui o conceito de herança e subtipos (ex: a classe `Book` estende `Product`). O modelo relacional de banco de dados não suporta o conceito de herança nativamente.

3. **Identity Mismatch (Descompasso de Identidade):**
   * **Problema:** Em um banco de dados, duas entidades são consideradas a mesma se tiverem a mesma *Primary Key (Chave Primária)*. Em Java, existem dois conceitos de identidade: a **identidade do objeto** (endereço de memória, checado com `==`) e a **igualdade do objeto** (valores internos, checados implementando o método `.equals()`).

4. **Associations Mismatch (Descompasso de Associações):**
   * **Problema:** Em Java, as associações são feitas via *referências de objetos* e podem ser **direcionais/bidirecionais** (ex: um Livro conhece o Autor e o Autor conhece o Livro). No banco de dados, associações são feitas usando *Foreign Keys (Chaves Estrangeiras)*, que não possuem o conceito de "direção" da mesma forma.

5. **Data Navigation Mismatch (Descompasso de Navegação de Dados):**
   * **Problema:** Em Java, acessamos dados "caminhando" pelo grafo de objetos (ex: `book.getPublisher().getName()`). Fazer essa mesma navegação iterativa no banco de dados exigiria executar muitas consultas SQL ineficientes. No banco relacional, a forma eficiente de buscar dados interconectados é usando `JOINs` para reduzir as chamadas de rede.

---

## Aula 3 e 6: Object Relational Mapping (ORM) na Prática
Nestes laboratórios, foi utilizado o projeto `bookstore` para mostrar a dor e a complexidade de gerenciar a persistência manualmente com **JDBC puro** (Java Database Connectivity), destacando por que usar ferramentas de **ORM** (como o Hibernate) é tão vital.

### O Modelo (Bookstore)
Para demonstrar o problema, as aulas trabalharam com três entidades de dados e suas respectivas tabelas no banco MySQL.

**Sintaxe de criação do Banco (SQL ensinado):**
```sql
CREATE TABLE PUBLISHER (
	CODE VARCHAR(4) NOT NULL,
	PUBLISHER_NAME VARCHAR(100) NOT NULL,
	PRIMARY KEY (CODE)
);

CREATE TABLE BOOK (
	ISBN VARCHAR(50) NOT NULL,
	BOOK_NAME VARCHAR(100) NOT NULL,
	PUBLISHER_CODE VARCHAR(4),
	PRIMARY KEY (ISBN),
	FOREIGN KEY (PUBLISHER_CODE) REFERENCES PUBLISHER (CODE)
);

CREATE TABLE CHAPTER (
	BOOK_ISBN VARCHAR(50) NOT NULL,
	CHAPTER_NUM INT NOT NULL,
	TITLE VARCHAR(100) NOT NULL,
	PRIMARY KEY (BOOK_ISBN, CHAPTER_NUM),
	FOREIGN KEY (BOOK_ISBN) REFERENCES BOOK (ISBN)
);
```

As classes equivalentes no Java (POJOs - Plain Old Java Objects) possuíam os mesmos campos (ex: classe `Book` com atributos `isbn`, `name`, objeto `Publisher` e uma lista `List<Chapter>`).

### O Problema ao utilizar JDBC Puro
Ao tentar salvar ou buscar um livro e todos os seus capítulos e publicadoras de uma vez (o *Grafo de Objetos*), o código escrito manualmente com JDBC ficava extremamente grande e repetitivo.

**Sintaxe do JDBC ensinada (Salvando dados de forma manual):**
```java
// É necessário criar queries específicas e inserir cada campo na mão
PreparedStatement stmt = connection.prepareStatement("INSERT INTO PUBLISHER (CODE, PUBLISHER_NAME) VALUES (?, ?)");
stmt.setString(1, book.getPublisher().getCode());	
stmt.setString(2, book.getPublisher().getName());			
stmt.executeUpdate();
stmt.close();

// Repetir a mesma lógica manual de JDBC para BOOK e para cada CHAPTER do Livro
```

**Problemas identificados dessa abordagem:**
1. **Linguagem extra:** O desenvolvedor Java precisa dominar perfeitamente SQL.
2. **Muitas consultas:** Para salvar um único Grafo de Objeto complexo, dezenas de *statements* precisam ser escritos manualmente.
3. **Código "Copia e Cola" (Boilerplate):** Processo exaustivo de ler variáveis de um objeto e mapeá-las uma por uma em métodos `setString()`, `setInt()`, etc.
4. **Locking com o Banco de Dados:** O código SQL puro escrito é amarrado à sintaxe do SGBD utilizado (MySQL). Mudar para PostgreSQL exigiria reescrever as queries.

### A Solução: Object Relational Mapping (ORM)
O Mapeamento Objeto-Relacional abstrai e resolve todos esses problemas. Através do ORM (seja por anotações no código ou por XML), você apenas instrui como a classe Java se liga com a tabela e suas colunas.

Uma vez feito o mapping, para persistir todo o grafo do Livro (Livro, Publicadora, Capítulos), com Hibernate você usaria um simples método nativo:
```java
// O ORM assume o trabalho pesado
session.save(book);
```
O ORM assume a complexidade de abrir a conexão, converter o grafo para os relacionamentos do SGBD, criar e disparar todas as cláusulas `INSERT` ou `JOINs` necessárias, além de lidar automaticamente com as chaves estrangeiras.

---

## Aula 7: Lab Exercise
A última aula da sessão reforçou os conceitos e convidou à execução do código baixado:
- Executar os códigos manualmente.
- Confirmar a persistência correta no MySQL via queries de verificação.
- Foi feita uma menção para prestar muita atenção na dependência e driver correto baseado na versão do MySQL (se é 5.x ou 8.x).
