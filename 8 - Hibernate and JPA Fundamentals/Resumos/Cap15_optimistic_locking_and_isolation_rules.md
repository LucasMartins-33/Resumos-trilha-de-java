# Capítulo 15: Optimistic Locking, Pessimistic Locking e Transaction Isolation Rules

## 1. O Problema da "Atualização Perdida" (Lost Update)

Imagine o cenário clássico de uma aplicação multiusuário (uma "Conversação" que dura múltiplas transações):
1. O **Usuário A** carrega o `Guide` de ID=2 (Salário = 2500) para editar na tela.
2. Segundos depois, o **Usuário B** também carrega o mesmo `Guide` de ID=2 na sua tela.
3. O **Usuário B** é rápido, altera o salário para **4000** e clica em salvar. O banco de dados é atualizado para 4000.
4. O **Usuário A** termina de pensar, altera o salário do seu lado para **3000** e clica em salvar.

**O que acontece?** A transação do Usuário A irá sobrescrever a alteração do Usuário B sem nem perceber. O banco salvará 3000. A alteração do Usuário B foi **completamente perdida**. Em banco de dados, chamamos isso de regra do *"Último Commit Vence"* (Last commit wins). Em sistemas financeiros ou de negócio, isso é um desastre.

---

## 2. A Solução: Optimistic Locking (Versionamento)

Para prevenir "Lost Updates", o JPA utiliza uma estratégia chamada **Optimistic Locking** (Travamento Otimista). Nós a chamamos de "otimista" porque ela *não trava* o banco de dados fisicamente; ela pressupõe otimisticamente que colisões não acontecerão com frequência e, caso aconteçam, o sistema barra a atualização final.

### 2.1. Como implementar?
Tudo o que precisamos é adicionar uma coluna de versão (numérica) na tabela e mapeá-la usando a anotação `@Version`.

```java
@Entity
public class Guide {
	
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY) 
	private Long id;
	
	// Adicionando a coluna de versão (o banco de dados deve ter default 0)
	@Version
	private Integer version;

	private Integer salary;
    // ...
}
```

### 2.2. Como o Hibernate age sob o capô?
1. Quando a linha é criada, `version` começa em 0.
2. O **Usuário A** e o **Usuário B** carregam o objeto com `version = 0`.
3. O **Usuário B** salva. O Hibernate não faz um simples `UPDATE`. Ele checa a versão e a incrementa automaticamente: 
   `UPDATE guide SET salary = 4000, version = 1 WHERE id = 2 AND version = 0` (Sucesso!).
4. Quando o **Usuário A** tenta salvar, o Hibernate tenta:
   `UPDATE guide SET salary = 3000, version = 1 WHERE id = 2 AND version = 0`.
5. **A Mágica:** Como o banco já está na versão 1 (devido ao Usuário B), o banco avisa ao Hibernate que *zero linhas* foram afetadas.
6. O Hibernate entra em pânico, impede a transação e lança uma **`OptimisticLockException`** (causada por uma `StaleObjectStateException`). No código, você captura essa exception e avisa amigavelmente ao usuário: *"Os dados que você está tentando alterar foram modificados por outra pessoa recentemente"*.

---

## 3. Bulk Operations (executeUpdate) vs Versionamento

As operações em massa, como um `UPDATE` massivo usando a linguagem JPQL, **ignoram o contexto de persistência** e vão direto para o banco. Por irem direto para o banco, duas coisas perigosas acontecem:
1. O Hibernate **não executa os Callbacks de ciclo de vida** (ex: `@PostUpdate`).
2. O Hibernate **não atualiza a coluna `@Version` automaticamente**!

Para consertar a falta de atualização do `@Version` em atualizações em lote (Bulk Updates), você deve utilizar a palavra-chave proprietária do Hibernate `VERSIONED` dentro da sua JPQL:

```java
// O uso da palavra "versioned" fará o banco incrementar a coluna @Version de cada User modificado.
int updatedUsers = em.createQuery(
    "update versioned User u set u.level = :level where u.numOfPosts >= :nop and u.isActive = :isActive")
    .setParameter("level", 5)
    .setParameter("nop", 100)
    .setParameter("isActive", true)
    .executeUpdate();
```

---

## 4. Pessimistic Locking (Travamento Físico)

Se o *Optimistic Locking* não usa travas, o **Pessimistic Locking** usa travas reais no SGBD (ex: cláusulas `FOR UPDATE` no SQL).
Ele é usado quando você está fazendo operações críticas na mesma transação, como calcular um relatório gigantesco (ex: folha de pagamentos) e você não quer que ninguém altere os salários do banco de dados enquanto seu `for-loop` ainda está somando e gerando o resultado, para evitar inconsistência de leitura.

```java
// READ Lock: Os outros usuários até conseguem ler, mas não conseguem modificar esse dado enquanto eu não terminar minha transação.
List<Object[]> resultList = em.createQuery("select guide.name, guide.salary from Guide as guide")
		.setLockMode(LockModeType.PESSIMISTIC_READ)
		.getResultList();

// WRITE Lock: Impede que outros usuários até mesmo LEIAM o dado enquanto minha transação não comitar.
List<Object[]> resultList = em.createQuery("select guide.name, guide.salary from Guide as guide")
		.setLockMode(LockModeType.PESSIMISTIC_WRITE)
		.getResultList();
```
*Atenção:* Travas pessimistas seguram recursos do banco. Mantenha as transações minúsculas, caso contrário a escalabilidade da sua aplicação irá despencar.

---

## 5. Transaction Isolation Rules (Regras de Isolamento)

O nível de isolamento de uma transação define "o quanto uma transação enxerga o que as outras estão fazendo". Quanto maior o nível, maior a integridade, porém menor a velocidade (escalabilidade).

Temos 4 níveis (do mais rígido para o mais brando):
1. **Serializable (O mais Rígido):** Transações rodam uma atrás da outra de forma serial (em fila). Integridade de 100%, mas péssimo para performance.
2. **Repeatable Read:** (Padrão no MySQL). Dentro da sua transação, se você ler a mesma linha duas vezes, ela sempre terá o mesmo valor (mesmo que outro usuário tenha feito um commit no meio tempo). O risco aqui é o *"Phantom Read"* (leitura fantasma), que é quando uma nova linha inserida por outro usuário aparece no meio da sua consulta.
3. **Read Committed:** (Padrão no Oracle). Você só enxerga coisas que já receberam commit de outros usuários. Se o seu loop demorar 5 minutos, você pode consultar o mesmo registro 2 vezes e obter salários diferentes porque alguém aplicou um commit no meio do processo (*Non-repeatable read*).
4. **Read Uncommitted (O mais Brando):** Extremamente perigoso. Você consegue ler dados de transações que ainda nem receberam commit (*Dirty Read*). Se a outra pessoa der Rollback, seu sistema agiu baseado num dado que nunca existiu de fato.

### Como configurar no Hibernate (`persistence.xml`)
Você pode instruir o Hibernate/Driver JDBC sobre qual nível de isolamento usar na conexão usando numerais de mapeamento: `1` (Read Uncommitted), `2` (Read Committed), `4` (Repeatable Read) e `8` (Serializable).

```xml
<!-- Exemplo: setando para READ COMMITTED -->
<property name="hibernate.connection.isolation" value="2"/>
```
