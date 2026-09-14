# Capítulo 14: Merging Detached Objects (e a armadilha do equals/hashCode)

## Introdução
Em aplicações reais com múltiplos usuários (como sistemas web), é uma péssima prática manter uma conexão JDBC e uma transação de banco de dados abertas enquanto o usuário pensa, preenche um formulário na tela e clica em "Salvar". O correto é carregar os dados, fechar a transação, enviar os objetos para a tela e, depois, receber os objetos modificados para salvar.

Neste intervalo, esses objetos perdem o vínculo com o JPA. Eles se tornam **Detached Objects (Objetos Desanexados)**. Este capítulo foca em como reanexar (Merge) esses objetos e as terríveis armadilhas de identidade do Java associadas a eles.

---

## 1. Como funciona o Merge (Mesclagem)

Imagine o seguinte fluxo:
1. O `EntityManager 1` busca o Guia de ID=2. Transação é comitada e o `em1` é fechado.
2. O Guia agora está *Detached* (desanexado).
3. Alteramos o salário do Guia via código: `guide.setSalary(2500)`.
4. Abrimos um novo `EntityManager 2` e tentamos salvar a alteração.

Para salvar um objeto desanexado, **não usamos o `em.persist()`**. Nós usamos o **`em.merge()`**.

```java
EntityManager em2 = emf.createEntityManager();
em2.getTransaction().begin();

Guide mergedGuide = em2.merge(guide); // O pulo do gato!

em2.getTransaction().commit();
em2.close();
```

### 1.1. O que acontece por baixo dos panos no `merge()`?
O método `merge` não executa um `UPDATE` imediatamente. Ele faz um trabalho cirúrgico:
1. Ele verifica se já existe um Guia de ID=2 gerenciado pelo `em2`.
2. Como não existe, ele executa um `SELECT` no banco para carregar os dados originais e anexa esse novo objeto ao contexto.
3. Ele **copia** meticulosamente os dados do seu objeto *Detached* (que estava com salário 2500) para dentro deste novo objeto gerenciado.
4. Ao dar o `commit`, o mecanismo de *Dirty Checking* do JPA detecta a alteração e emite o `UPDATE`.

### 1.2. O Perigo de esquecer o Cascade no Merge
Se o nosso `guide` possuía uma coleção de Estudantes, e nós alteramos o salário do guia E o nome de um dos estudantes, chamar `em.merge(guide)` **NÃO atualizará o estudante por padrão!** Apenas o objeto-pai sofrerá o merge.
Para que as alterações nos filhos também sejam salvas, você é **obrigado** a ligar o Cascade de Merge no mapeamento:

```java
@OneToMany(mappedBy="guide", cascade={CascadeType.MERGE}) // CRÍTICO para salvar os filhos!
private Set<Student> students = new HashSet<Student>();	
```

---

## 2. A Armadilha do equals() e hashCode() com Detached Objects

O problema mais clássico do JPA acontece quando você tenta colocar objetos desanexados dentro de uma coleção `Set` (que não aceita itens duplicados).

Se você buscar o Estudante de ID=4 no `em1`, e depois buscar o mesmo Estudante de ID=4 no `em2`, você terá **dois objetos diferentes na memória do Java**, mesmo que representem a mesma linha exata no banco de dados.

Se você tentar colocar os dois num `HashSet`, o Java usará o método `equals()` padrão (que compara endereços de memória). Como os endereços são diferentes, **a coleção aceitará os dois!** Você terá dados duplicados ferindo a lógica de negócio.

### 2.1. A Solução: Sobrescrever equals e hashCode
A resposta óbvia é sobrescrever esses métodos na classe `Student`. Mas há uma **Regra de Ouro** que você nunca deve quebrar.

**O que NÃO usar: O Identificador do Banco (`@Id`)**
NUNCA implemente o `hashCode` e `equals` usando a chave primária (`Long id`). 
**Por quê?** Quando você cria um objeto novo na aplicação, o `id` dele é nulo (`null`). Se você o colocar num `HashSet`, ele será guardado num "balde" (*bucket*) baseado no valor Nulo. Ao chamar `em.persist(student)`, o banco gera o ID e o injeta no objeto. O hashCode do objeto muda no meio do caminho! A coleção do Java vai "quebrar" e não conseguirá mais localizar o seu objeto lá dentro (o `set.contains()` passará a retornar `false`).

**O que USAR: Chaves de Negócio (Business Keys)**
A implementação perfeita utiliza chaves naturais/de negócio que nunca mudam e que já nascem com o objeto (ex: `enrollmentId`, `cpf`, `isbn`, `email`).

```java
import org.apache.commons.lang3.builder.EqualsBuilder;
import org.apache.commons.lang3.builder.HashCodeBuilder;

@Entity
public class Student {
	
	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY) 
	private Long id; // JAMAIS use isso no equals/hashcode
	
	@Column(name="enrollment_id", nullable=false, unique=true)
	private String enrollmentId; // USE ISTO! (Chave de Negócio)
	
	// Precisamos do getter para o Hibernate lidar com os Proxies corretamente
	public String getEnrollmentId() {
		return enrollmentId
	}

	@Override
	public int hashCode() {
		 // Usando o método getter ao invés da propriedade direto
		return new HashCodeBuilder().append(this.getEnrollmentId()).toHashCode();
	}
	
	@Override
	public boolean equals(Object obj) {
		if (!(obj instanceof Student)) return false;
		Student other = (Student) obj;
		
		// Usando os métodos getters de ambas as instâncias
		 return new EqualsBuilder()
                    .append(this.getEnrollmentId(), other.getEnrollmentId())
                    .isEquals();
	}
}
```
Seguindo esse padrão, objetos Transient e Detached conviverão perfeitamente e suas coleções Java e de persistência estarão sempre 100% síncronas.
