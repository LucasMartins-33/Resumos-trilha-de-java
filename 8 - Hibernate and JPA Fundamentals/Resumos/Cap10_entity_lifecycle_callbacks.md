# Capítulo 10: Entity Lifecycle Callbacks

## Introdução
Neste capítulo, exploraremos os **Entity Lifecycle Callbacks** (Callbacks de Ciclo de Vida da Entidade). O JPA nos permite interceptar e reagir a eventos específicos que ocorrem durante o ciclo de vida de uma entidade dentro do *Persistence Context* (Contexto de Persistência).

Seja para calcular um campo temporário assim que a entidade for carregada do banco, ou para preencher automaticamente uma data de "última modificação" antes de um `UPDATE`, os *callbacks* são ferramentas essenciais.

Adicionalmente, entenderemos como o JPA lida com atributos transientes (não persistidos) através da anotação `@Transient`.

---

## 1. Atributos Não-Persistidos (`@Transient`)
Nem todo atributo da nossa classe Java precisa se tornar uma coluna no banco de dados. Um exemplo clássico é a **Idade** (`age`). Como a idade muda com o tempo, é uma boa prática persistir apenas a **Data de Nascimento** (`dateOfBirth`) e calcular a idade em tempo de execução.

Para dizer ao JPA que ignore um atributo, usamos a anotação `@Transient`.

**Exemplo Prático:**
```java
@Entity
public class Person {
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY) 
    private Long id;
    
    @Column(name="date_of_birth")
    private LocalDate dateOfBirth;
    
    @Transient // Não será criado no banco de dados
    private Integer age;
    
    //...
}
```
Ao invocar `em.persist(person)`, o Hibernate ignorará solenemente o campo `age`.

---

## 2. Callback Annotations: Interceptando o Ciclo de Vida

No JPA, um método de *callback* dentro da entidade deve obedecer a três regras estritas:
1. Não pode receber nenhum argumento (zero parâmetros).
2. O retorno deve ser `void`.
3. Não pode ser `static` ou `final`.

Vejamos os 7 tipos de *Callbacks* disponíveis:

### 2.1. `@PostLoad`
É acionado no exato momento em que uma entidade termina de ser carregada do banco de dados para dentro do Contexto de Persistência (seja por um `em.find()`, execução de Query ou um `em.refresh()`).

**Cenário de uso:** Calcular a idade automaticamente assim que o objeto é lido do banco!

```java
@PostLoad
public void calculateAge() {
    // LocalDate vem do pacote java.time (Java 8), perfeitamente suportado pelo Hibernate 5+
    this.age = Period.between(this.dateOfBirth, LocalDate.now()).getYears();
    System.out.println("@PostLoad invocado. A idade calculada foi: " + age);   
}
```

### 2.2. `@PrePersist` e `@PostPersist`
* **`@PrePersist`:** Executado no momento em que chamamos `em.persist()` ou passamos uma entidade nova (transiente) para um `em.merge()`. Ocorre **antes** que o SQL `INSERT` seja de fato emitido para o banco de dados.
* **`@PostPersist`:** Executado **depois** que o SQL `INSERT` é emitido para o banco. 
    * *Nota:* Lembre-se do Capítulo 08: Se sua entidade usar `GenerationType.IDENTITY`, o `INSERT` ocorrerá na mesma linha do `em.persist()`. Se usar `SEQUENCE`, o `INSERT` só ocorrerá no `commit` (flush). O momento do `@PostPersist` acompanha essa emissão do SQL!

**Exemplo:** Marcar a data e hora em que o registro foi criado:
```java
@Column(name="last_update")
private LocalDateTime lastUpdate;

@PrePersist
public void setCreationDate() {
    this.lastUpdate = LocalDateTime.now();
}
```

### 2.3. `@PreUpdate` e `@PostUpdate`
Acionados quando o Hibernate detecta que o estado do objeto persistente ficou sujo (*dirty*) e precisa ser sincronizado com o banco.
* **`@PreUpdate`:** Executado no momento do *Flush*, pouco antes do comando `UPDATE` ir para o banco.
* **`@PostUpdate`:** Executado logo após o término do `UPDATE` no banco.

*Dica: Você pode encadear as anotações no mesmo método!*
```java
@PrePersist
@PreUpdate
public void setLastUpdate() {
    this.lastUpdate = LocalDateTime.now();
}
```

### 2.4. `@PreRemove` e `@PostRemove`
Acionados quando invocamos `em.remove(entidade)`.
* **`@PreRemove`:** Antes do SQL `DELETE`. Útil para fazer uma verificação de segurança ou limpar dependências na memória.
* **`@PostRemove`:** Após o SQL `DELETE`. Útil para engatilhar um alerta de auditoria após a remoção bem-sucedida.

---

## 3. Desacoplando Callbacks usando `@EntityListeners`

No exemplo anterior, misturamos a lógica de log e auditoria diretamente dentro da nossa classe de domínio (`Person` ou `Message`). Mas e se quisermos fazer isso de fora da entidade?

Podemos usar **Entity Listeners**. 

Um *Entity Listener* é uma classe Java comum, sem anotações de entidade, que contém os métodos de callback. As regras mudam levemente:
1. O método de *callback* **pode (e deve) receber um parâmetro `Object`** representando a instância da entidade que sofreu o evento.
2. A classe Listener **deve possuir um construtor público sem argumentos** (Padrão/Default Constructor).

**Passo 1: Criando as classes Listener**
```java
package listener;

import java.time.LocalDateTime;
import entity.Message;
import jakarta.persistence.PrePersist;
import jakarta.persistence.PreUpdate;
import jakarta.persistence.PostPersist;

public class LastUpdateListener {
    @PrePersist
    @PreUpdate
    public void setLastUpdate(Object obj) {
        if (obj != null && obj instanceof Message) {
            Message msg = (Message) obj;
            msg.setLastUpdate(LocalDateTime.now());
        }
    }
}

public class NotificationListener {
    @PostPersist
    public void notifyAdmin(Object entityInstance) {
        System.out.println("Enviando uma mensagem de notificação após o INSERT...");
    }
}
```

**Passo 2: Anotando a Entidade**
Para vincular a entidade aos listeners, usamos a anotação `@EntityListeners` informando um array de classes Listener. 

```java
@Entity
@EntityListeners({LastUpdateListener.class, NotificationListener.class}) // Suporta múltiplos listeners!
public class Message {
    
    @Id
    @GeneratedValue(strategy=GenerationType.IDENTITY) 
    private Long id;
    
    private String text;	
    
    @Column(name="last_update")
    private LocalDateTime lastUpdate;	
    
    // Getters e setters omitidos...
}
```
Com o uso de `@EntityListeners`, conseguimos separar as regras de negócio de domínio limpo (dentro da entidade) das preocupações auxiliares do sistema, tais como auditoria de datas ou disparos de notificações!
