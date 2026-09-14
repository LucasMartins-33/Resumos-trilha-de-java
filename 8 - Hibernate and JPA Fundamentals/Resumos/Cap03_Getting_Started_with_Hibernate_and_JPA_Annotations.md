# Capítulo 03: Getting Started with Hibernate and JPA Annotations (Versão Estendida e Detalhada)

Este capítulo marca a introdução prática ao uso do **Hibernate** em conjunto com as **Anotações do JPA (Java Persistence API)**. A partir daqui, nos afastamos completamente do JDBC manual e começamos a deixar o trabalho pesado de persistência a cargo do ORM (Object Relational Mapping).

Abaixo, você encontrará um detalhamento profundo de todas as aulas do Capítulo 03, construído para servir como material de consulta avançada.

---

## 1. O que é o Hibernate e por que usá-lo?
*(Aulas 8 e 9)*

Desenvolvedores Java estão acostumados a criar **POJOs** (Plain Old Java Objects) com propriedades e métodos de acesso (Getters e Setters). Quando precisamos enviar esses objetos para um Banco de Dados Relacional, o desenvolvedor é tradicionalmente forçado a "falar outra língua", que é o SQL. 

O Hibernate atua exatamente aí: ele **cria o código SQL dinamicamente em runtime** para Salvar, Atualizar, Deletar e Consultar dados (CRUD).
Além de abstrair o SQL, o Hibernate também trata de:
* **Impedance Mismatch:** Resolve os 5 descompassos clássicos entre orientação a objetos e bancos relacionais.
* **Transações e Concorrência:** Gerencia como os dados são isolados quando várias interações estão acontecendo ao mesmo tempo.
* **Performance:** Possui cache e otimização de instruções (ex: envio em batch).

### O Papel do Construtor Vazio (Regra de Ouro)
Um detalhe crucial levantado nas aulas de manipulação de objetos é que **Toda classe Entidade DEVE possuir um construtor sem argumentos (no-argument constructor)**. 
O motivo? O Hibernate precisa instanciar os seus objetos vindo do banco através da tecnologia de *Java Reflection*. Se você não fornecer o construtor vazio, o Hibernate jogará uma `InstantiationException` em tempo de execução ao tentar buscar algo via `.find()` ou `.get()`.

---

## 2. Hello World: Configurando o Ambiente e a Entidade
*(Aulas 9 e 10)*

### A. O Arquivo `hibernate.cfg.xml` e a Sessão
O núcleo de configuração da aplicação é o arquivo `hibernate.cfg.xml`. Ele fica na raiz do Classpath e diz ao Hibernate: como conectar, qual sotaque SQL usar (Dialeto), se deve imprimir o SQL e quais Entidades estão mapeadas.

```xml
<hibernate-configuration>
    <session-factory>
        <!-- Dados de Conexão com o Banco -->
        <property name="connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="connection.url">jdbc:mysql://localhost:3306/hello-world</property>
        <property name="connection.username">root</property>
        <property name="connection.password">password</property>

        <!-- MySQLDialect otimiza a sintaxe gerada de acordo com as peculiaridades do MySQL -->
        <property name="dialect">org.hibernate.dialect.MySQLDialect</property>
        
        <!-- Onde avisamos que a classe entity.Message precisa ser gerenciada -->
        <mapping class="entity.Message"/> 
    </session-factory>
</hibernate-configuration>
```

A partir dessa configuração, criamos uma **SessionFactory**.
> **Ponto de Arquitetura:** Criar a `SessionFactory` é extremamente pesado para a aplicação (processo custoso de leitura de metadados e configuração inicial). Por isso, na nossa classe utilitária `HibernateUtil`, ela é criada apenas **uma única vez (Single Instance)** e compartilhada pelo resto do sistema inteiro. A partir dessa fábrica única, nós abrimos inúmeras "Sessions" leves.

### B. Mapeando a Classe com Anotações JPA
O projeto começou utilizando arquivos `XML` arcaicos (`.hbm.xml`) para o mapeamento, apenas como introdução histórica. Depois, o curso o descarta definitivamente em prol do JPA Annotations.

```java
package entity;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;

@Entity // Torna a classe uma "Entidade Persistente" gerenciada pelo Hibernate
@Table(name="message") // Opcional se o nome da tabela fosse exatamente o nome da classe
public class Message {

	@Id // Indica o atributo que serve de Chave Primária
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="ID")	
	private Long id;

	@Column(name="TEXT") // Opcional se a coluna chamasse exatamente "text"
	private String text;
	
	public Message() {} // Nosso construtor vazio obrigatório
	
	public Message(String text) {
		this.text = text;
	}
	// ... Getters e Setters
}
```

#### Atualizações de Versões do Hibernate (Notas do Autor nas Aulas 11, 12, e 13)
* **Mudança no `GenerationType.AUTO`:** Antes do Hibernate 5.x, usar `AUTO` utilizava a estratégia de `IDENTITY` por trás. A partir da versão 5.x, o `AUTO` passou a tentar usar `SEQUENCE`. O MySQL costuma lidar melhor com `IDENTITY` (auto_increment). Por isso, se estiver do Hibernate 5 em diante, mude explicitamente para `GenerationType.IDENTITY`.
* **Jakarta EE:** A partir do Hibernate 6, a especificação Java migrou de pacote. Os imports deixaram de ser `javax.persistence.*` e passaram a ser `jakarta.persistence.*`.
* **Java Mínimo:** Hibernate 6 exige Java 11. Hibernate 7 exige Java 17 no mínimo.
* **Depreciações:** O método `.save()` foi depreciado pelo JPA em prol do `.persist()`. O método `.get()` em prol do `.find()`, `.delete()` por `.remove()`, entre outras.

---

## 3. Manipulando Objetos e a Natureza das Transações
*(Aulas 14, 17, 18 e 19)*

O Hibernate trabalha com o conceito de **Unit of Work** (Unidade de Trabalho) controlada pelas transações e Sessão. 

### O Ciclo de Vida do Objeto (Os Estados)
Ao trabalhar com instâncias no Java atreladas a banco de dados, todo objeto passa por "Estados". 
No laboratório de "Object States", o professor listou as transições exatas:

1. **Transient (Transitório):** Você dá o comando `Message msg = new Message("Hello");`. Ele existe apenas na memória. O banco nem sabe disso e não existe Primary Key nele (O id é `null`).
2. **Persistent (Persistente):** Você manda o Hibernate tomar conta dele via `session.persist(msg);`.
   * **O que acontece nos bastidores?** Um ID é imediatamente gerado e designado ao objeto na memória (o `id` deixa de ser `null` e vira `1`, por exemplo).
   * **Mas a tabela ainda está vazia!** Se você pausar em Debug antes do `commit()`, o banco **ainda não tem os dados na tabela visíveis**. Isso ocorre pois a execução está presa dentro da transação JDBC do banco de dados, protegendo a atomicidade. Os dados de fato se tornam finais quando a Transação faz um commit (`txn.commit()`).
3. **Detached (Desanexado):** Quando a `session.close()` é chamada, aquele objeto na sua tela ainda existe na memória, e existe no banco, **mas eles perderam a sincronia**. O Hibernate não está mais os gerenciando juntos.
4. **Removed (Removido):** O objeto foi escalado para ser deletado na hora do próximo commit.

### Atualizando Objetos: Automatic Dirty Checking vs Merge()
As regras de atualização de um dado provaram ser um dos pontos mais importantes da aula. Existem 2 formas de atualizar.

**Forma 1: O Caminho Feliz (Automatic Dirty Checking)**
Se você carrega um objeto em uma sessão aberta e modifica ele através de Setters, o Hibernate descobre automaticamente.
```java
Message message = session.find(Message.class, 2L); // O Hibernate sabe a partir de agora o estado do id=2
message.setText("Hello Automatic Dirty Checking!"); // Você mudou algo. Ele sujou a entidade.

session.getTransaction().commit(); // Mágica: O Hibernate sabe que o objeto foi modificado e DELEGA O UPDATE SOZINHO!
```
Você não precisou chamar nenhum comando explícito de Update ou Save! Isso é o **Automatic Dirty Checking**. O Hibernate compara o estado inicial com o estado no final da transação.

**Forma 2: O Problema do Objeto Detached e o método `.merge()`**
Na Aula 19, o professor provoca a seguinte situação (O pesadelo comum dos iniciantes):
1. O Objeto 3 (id=3) é carregado na Sessão 1. A sessão 1 é fechada.
2. Na memória (Estado Detached), seu atributo de texto é mudado para `Hi`. 
3. Você abre a Sessão 2 e tenta sincronizar esse estado detached de volta usando `session2.update(message)`.

*O que dá errado?* Se a Sessão 2 *já tivesse* carregado o Objeto de `id=3` previamente, chamar `update()` com um objeto Detached (mesmo de id=3) vai soltar a exceção violenta: `NonUniqueObjectException`. A sessão fica confusa, dizendo: "Eu já tenho um objeto com ID=3 na minha memória, você está me jogando outro em cima!".

*A Solução Definitiva:* **`.merge(message)`**
```java
Message message2 = session2.find(Message.class, 3L);
session2.merge(message); // Mescla as diferenças de "message" (detached) para "message2" (persistent) com segurança.
session2.getTransaction().commit();
```
O `.merge()` diz ao Hibernate: "Pegue os valores do meu objeto Detached e copie por cima das propriedades do objeto que você tem na memória, combinando os dois de forma segura". Por isso, a partir do Hibernate 6, `update()` foi removido e pede-se para sempre usar `merge()`.

---

## 4. O Sistema de Logging Completo: Espionando o Hibernate
*(Aulas 15 e 16)*

O Hibernate trabalha muito nos bastidores. Quando as coisas dão errado, ou quando queremos monitorar performance, devemos ativar os Logs. O Hibernate 4+ abstraiu seus logs e sugere usar pacotes profissionais como o **Log4j** via `log4j.properties`.

**Visualizando os SQLs e as Variáveis:**
Tirar o `show_sql` do arquivo de configuração e deixar a cargo do log4j:
```properties
# Liga a impressão dos comandos SQL executados:
log4j.logger.org.hibernate.SQL=ALL

# (O MAIS IMPORTANTE) Substitui as interrogações '?' dos parâmetros pelos valores reais sendo processados:
log4j.logger.org.hibernate.orm.jdbc.bind=TRACE
```

**Ativando o "Raio-X" da Sessão (Estatísticas Internas):**
Se você deseja analisar gargalos (como quanto tempo uma transação durou, se a query atingiu o Cache L2 ou se bateu direto no JDBC e precisou alocar recurso pesado), usamos a estatística.
Ligue a feature e acompanhe o pacote interno:
```properties
# (Pode ir também no hibernate.cfg.xml: hibernate.generate_statistics=true)

# Habilita o print detalhado (nanosegundos, conexões usadas, impacto de cache L2):
log4j.logger.org.hibernate.engine.internal.StatisticalLoggingSessionEventListener=INFO
```
*(Cuidado: Nunca habilite estatísticas ou logs em trace level em produção, devido a grave queda de performance e geração excessiva de arquivos de log).*
