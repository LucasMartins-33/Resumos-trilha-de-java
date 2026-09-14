# Capítulo 5: Primeiros Passos com JPA (Getting Started with JPA)

Olá, aluno(a)! Neste capítulo, vamos dar um passo importantíssimo no curso: iniciar nossos estudos detalhados sobre a **Java Persistence API (JPA)**, ou Jakarta Persistence, e como ela atua em conjunto com o Hibernate. Até então, vínhamos usando anotações da JPA em nossas entidades (`@Entity`, `@Table`, `@Id`, etc.), mas utilizando as classes nativas do Hibernate para persistência (como `Session` e `SessionFactory`). 

A partir de agora, mudaremos essa abordagem! Veremos como programar voltado para as interfaces padrão do Java e exploraremos a fundo o ciclo de vida dos objetos, o conceito de Contexto de Persistência e como tirar proveito do cache. 

Prepare-se para aprender como essas APIs operam por baixo dos panos!

---

## 1. O que é a JPA e a sua Relação com o Hibernate

A **Java Persistence API (JPA)** é, em sua essência, **apenas uma especificação**. Ela fornece diretrizes e um conjunto de interfaces para acessar, persistir e gerenciar dados entre objetos Java e um banco de dados relacional. Por ser apenas uma especificação, ela *não possui código funcional para executar a persistência*. Ela necessita de uma implementação.

O **Hibernate**, que você já conhece, é um framework completo de mapeamento objeto-relacional (ORM) que possui a sua própria API nativa (aquela que usamos até o Capítulo 4). No entanto, o Hibernate também fornece uma **implementação completa** da especificação JPA. Portanto, o Hibernate atua como o que chamamos de **Provedor JPA (JPA Provider)**.

### Por que programar para a JPA em vez do Hibernate Nativo?

A grande vantagem de se usar a JPA está no fato de estarmos programando voltado para **interfaces**. Se você escrever todo o seu código focando exclusivamente na API da JPA (`EntityManager`, etc.), e num futuro precisar trocar o provedor ORM (por exemplo, trocar o Hibernate pelo EclipseLink por motivos de performance ou exigências de projeto), poderá fazer isso praticamente sem alterar o seu código fonte.

Porém, o Hibernate é extremamente poderoso e oferece funcionalidades que vão muito além da especificação JPA (os recursos "não-padrão" ou *non-JPA features*). Você perde o acesso a eles se usar apenas JPA? **Não!**
A JPA fornece o método `unwrap()`. Através dele, você consegue extrair a `Session` do Hibernate por trás das cortinas do `EntityManager` da JPA e assim usar recursos específicos e poderosos da API nativa, se julgar estritamente necessário.

---

## 2. O Hibernate como Provedor JPA

Vamos começar a substituir o código fortemente acoplado à API nativa do Hibernate pelo código em JPA. Na JPA, os papéis mudam de nome e de comportamento, embora sejam semanticamente equivalentes:

* **`Session`** (Hibernate) vira **`EntityManager`** (JPA).
* **`SessionFactory`** (Hibernate) vira **`EntityManagerFactory`** (JPA).
* O arquivo de configuração **`hibernate.cfg.xml`** é substituído pelo padrão da JPA, o arquivo **`META-INF/persistence.xml`**.
* A nossa classe customizada `HibernateUtil` (que criamos para gerar sessões) não é mais necessária! A JPA fornece uma classe utilitária de fábrica nativa chamada `Persistence`.

### O Arquivo `persistence.xml`

A primeira mudança prática em nossos projetos será o arquivo de configuração, que na JPA deve obrigatoriamente residir na pasta `META-INF` no *classpath* (exemplo: `src/main/resources/META-INF/persistence.xml`).

Veja como é a sua sintaxe, usando a *Unidade de Persistência* (Persistence Unit):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="https://jakarta.ee/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             version="3.0"
             xsi:schemaLocation="https://jakarta.ee/xml/ns/persistence https://jakarta.ee/xml/ns/persistence/persistence_3_0.xsd">

    <!-- Definimos uma unidade de persistência com um nome, que será chamado no nosso código Java -->
    <persistence-unit name="hello-world" transaction-type="RESOURCE_LOCAL">
        
        <!-- (Opcional) Podemos declarar as entidades, mas a maioria dos provedores detecta automaticamente -->
        <class>entity.Message</class>
        
        <properties>
            <!-- Configurações de conexão (agora com prefixos de propriedades jakarta/hibernate) -->
            <property name="jakarta.persistence.jdbc.driver" value="com.mysql.jdbc.Driver" />
            <property name="jakarta.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/hello-world" />
            <property name="jakarta.persistence.jdbc.user" value="root" />
            <property name="jakarta.persistence.jdbc.password" value="password" />

            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQLDialect" />
            <property name="hibernate.hbm2ddl.auto" value="update" />
            <property name="hibernate.format_sql" value="true" />
        </properties>
    </persistence-unit>
</persistence>
```

**Detalhe de Transação (`transaction-type`):**
Você notou o `transaction-type="RESOURCE_LOCAL"`? Isso informa à API que **nós, os desenvolvedores**, assumimos a responsabilidade de gerenciar as transações e gerar o `EntityManager`. Caso nosso aplicativo estivesse rodando dentro de um servidor de aplicação (como o GlassFish ou JBoss), o Contêiner EJB assumiria o controle das instâncias. Neste caso, utilizaríamos o `transaction-type="JTA"` (Java Transaction API), o que tornaria as coisas automáticas (não detalharemos JTA aqui para não complicar neste momento).

### Criando Sessões com `EntityManager` na Prática

Agora, vamos ver como usar essa configuração em nosso cliente de teste:

```java
import entity.Message;
import jakarta.persistence.EntityManager;
import jakarta.persistence.EntityManagerFactory;
import jakarta.persistence.EntityTransaction;
import jakarta.persistence.Persistence;

public class HelloWorldClient {
    public static void main(String[] args) {
        // Criando a fábrica baseada no nome "hello-world" mapeado no persistence.xml
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello-world");
        
        // Criando o equivalente da "Session"
        EntityManager em = emf.createEntityManager();
        
        // Iniciar uma transação
        EntityTransaction txn = em.getTransaction();
        txn.begin();
        
        Message message = new Message("Hello World with JPA");
        
        // Método persist() equivalente ao save() do hibernate antigo
        em.persist(message);
        
        // Commit salva as transações pendentes no BD
        txn.commit();
        
        // Sempre lembre de fechar o EntityManager
        em.close();
    }
}
```

> [!NOTE]
> **Alterações Importantes no Hibernate a partir da Versão 5.2**
>
> Antes do Hibernate 5.2, havia uma clara distinção entre as APIs. O `EntityManager` possuía uma classe implementadora (`EntityManagerImpl`) que, por sua vez, guardava uma referência e delegava funções para a classe nativa `SessionImpl`.
> A partir do Hibernate 5.2+, essa arquitetura foi alterada. A própria interface `Session` nativa passou a **estender** diretamente a interface `EntityManager` da JPA.
> **Na prática:**
> Se você obtém a `Session` nativa hoje, ela herda os métodos do `EntityManager`. Por exemplo, antes da 5.2 o método de buscar objetos pela `Session` exigia *cast*: `(Message) session.get(Message.class, 1L)`. Hoje, você pode usar livremente o `session.find(Message.class, 1L)` (método advindo da JPA) sem realizar *Typecast*, o que deixa tudo mais polido.

---

## 3. O Ciclo de Vida e Estados de um Objeto

Um dos conceitos mais valiosos que você deve dominar ao trabalhar com ORM é o **Ciclo de Vida do Objeto**. Quando lidamos com a JPA, os objetos em memória transitam entre 4 (quatro) estados fundamentais:

1. **Transiente (Transient):**
   É o objeto recém-nascido através da palavra chave `new`.
   - **Característica:** Não está associado a nenhum banco de dados (o ID é nulo) e não está sendo gerenciado pelo `EntityManager`.
   - **Ciclo:** Se perder a referência no Java, será devorado pelo *Garbage Collector* normalmente e desaparecerá para sempre, sem afetar o banco.

2. **Persistente (Persistent / Managed):**
   É o objeto que agora tem uma identidade de banco de dados (ID preenchido) **E** está ativamente sendo gerenciado pelo `EntityManager`.
   - **Como entrar neste estado?** Sendo passado para o `em.persist()`, sendo carregado de uma busca (`em.find()`) ou resultante de uma query JPA.
   - **Ciclo:** Qualquer alteração no estado deste objeto (exemplo: dar um `.setText("Hi")`) é ativamente mapeada pelo JPA e salva no Banco de Dados através de um `UPDATE` no momento do `commit()`.

3. **Desanexado (Detached):**
   É um objeto que **tinha identidade no BD (era Persistente)**, mas o seu "elo de monitoramento" com o `EntityManager` foi cortado (exemplo: o `EntityManager` foi fechado `em.close()` ou chamamos explicitamente `em.detach(objeto)`).
   - **Característica:** Alterar os campos no Java deste objeto **NÃO fará** com que a JPA dispare operações `UPDATE` no BD. O ORM lavou as mãos sobre este objeto.
   - **Ciclo:** Sendo um objeto comum no Java, pode ser coletado pelo GC se perder as referências. Para sincronizá-lo de volta com o banco, é preciso executar o método `em.merge()`.

4. **Removido (Removed):**
   É um objeto persistente (que obrigatoriamente precisa estar sob gerenciamento) que foi sinalizado para deleção via `em.remove(objeto)`. Ao efetuar o commit, a JPA emitirá um `DELETE` no banco.

> [!WARNING]
> **Erro muito comum:**
> O método `remove()` exige que o objeto seja **Persistente** (Managed). Se você tentar apagar um objeto que esteja em estado **Detached**, o provedor lançará uma assustadora `IllegalArgumentException`, acusando que a instância desanexada não pode ser removida. Busque a entidade com `find()` e então remova-a, se necessário.

### O Comportamento do método `merge()`

O método `merge()` resolve o problema de sincronizar alterações feitas quando o objeto estava **Detached**. 
Quando você altera propriedades de um objeto desanexado e deseja mandar essas alterações para o Banco de Dados, você deve criar um NOVO `EntityManager` e rodar o `.merge()`.

**Como o Merge funciona internamente:**
1. A JPA verifica a sua memória local à procura de um objeto gerenciado com o mesmo ID.
2. Se não o encontra (já que abrimos um EntityManager novo, por exemplo), ela vai no banco de dados, dá um `SELECT` naquele ID e traz o registro mais recente como um novo Objeto **Persistente**.
3. O `merge()` então copia todos os dados (valores das variáveis) do seu objeto **Detached** para o novo objeto **Persistente**.
4. Ele devolve o objeto **Persistente** como retorno da função!

> [!IMPORTANT]
> A referência original continua sendo Detached!
> Se você fizer `em.merge(messageDetached);`, a sua variável `messageDetached` continua desanexada e sendo ignorada!
> Você deve capturar o retorno do merge, por exemplo: `Message msgManaged = em.merge(messageDetached);`. Apenas alterações na `msgManaged` serão captadas e salvas em BD!

---

## 4. Cache de Primeiro Nível (Contexto de Persistência)

Muitas vezes você lerá as palavras **Persistence Context (Contexto de Persistência)**. O que elas significam?
Podemos simplificar dizendo: O `EntityManager` atua como um representante do Contexto de Persistência. E o Contexto de Persistência age nos bastidores como um **Cache de Primeiro Nível (First-Level Cache)** e um observador de estado.

A ideia de um Cache é criar cópias dos dados (linhas de tabelas do BD) da memória persistente (relacional) para dentro da memória volátil da nossa JVM. Isso traz saltos gigantescos de performance. O escopo do Cache de Primeiro Nível é **restrito ao próprio `EntityManager`**.

### Dirty Checking Automático (Verificação Suja)

O Cache não só guarda memória, ele cuida dos rastreios. O mecanismo chamado "Automatic Dirty Checking" funciona assim:
Quando um objeto passa a ser gerenciado (ao dar um `find()`, por exemplo), a JPA tira uma **fotografia (Snapshot)** de todo o estado (dados das colunas) daquele objeto na memória RAM.
Ao chegar o momento de executar `txn.commit()`, o Hibernate percorre os objetos gerenciados, comparando o estado atual contra o seu estado fotográfico (Snapshot). Apenas se ele achar uma discrepância (se o dado ficar "Dirty", sujo), ele monta o `UPDATE` de SQL, enviando as modificações pro banco. Sem discrepâncias, não envia nada e poupa a rede!

* Dica Avançada: Tirar Snapshots de uma malha gigantesca de entidades complexas custa tempo e memória RAM. Você pode reescrever esse comportamento com a JPA (para otimização profunda) implementando a interface `CustomEntityDirtinessStrategy` informando quando um objeto de fato deve ser marcado como sujo.

* Dica Prática: Se quiser forçar a sincronização de tudo que ficou sujo e empurrar os INSERTS e UPDATES para o BD sem ter que dar commit naquele momento (para validar IDs gerados, regras do banco, etc), utilize o método `em.flush()`. O flush joga o fluxo pendente do cache para o banco de dados antes do commit.

### Vantagens do Persistence Context / First Level Cache

1. **Evita consultas repetitivas (Cache Real):**
   ```java
   Message m1 = em.find(Message.class, 7L); // JPA roda SELECT no DB. Grava objeto no cache.
   Message m2 = em.find(Message.class, 7L); // JPA acha ID 7 no cache! Retorna IMEDIATAMENTE sem rodar SELECT.
   ```

2. **Leituras Repetíveis (Repeatable Read):**
   Enquanto você está sob a mesma transação (mesmo EntityManager), você sempre obterá a sua cópia do cache. Mesmo que no BD um outro programa tenha alterado um campo concorrentemente, o seu código lerá sempre o mesmo dado (seu estado local inalterado da transação que iniciou), garantindo consistência até terminar o seu trabalho.

3. **Garantia de Identidade (Guaranteed Scope of Object Identity):**
   Não existirá mais do que 1 objeto Java gerenciado para a mesma linha do BD sob o mesmo Contexto de Persistência. Se você chamou 5 vezes um ID através do find() numa mesma sessão, todas as 5 variáveis **apontam para o exato mesmo endereço de memória**. Isso previne dores de cabeça como dois pedaços de código na mesma transação alterando cópias separadas do mesmo dado simultaneamente, gerando inconsistências no commit.

### Como Gerenciar o Cache de Primeiro Nível (Gerência de Memória)

Lembre-se: O cache armazena as cópias e não encolhe sozinho ao decorrer da sessão! Em sistemas longos com processamento em lote, você pode ter instabilidade de memória (*OutOfMemoryError*) se o cache inflar infinitamente. Você tem ferramentas à disposição:

* **`em.detach(objeto)`:** Remove pontualmente uma entidade do cache, deixando-a isolada.
* **`em.clear()`:** É o trator de limpeza! Remove absolutamente tudo do cache de primeiro nível. Nenhum objeto que estava persistente continuará persistente. Ideal ao processar lotes grandes e após chamar o `em.flush()`, para limpar memória e recomeçar os trabalhos em branco na mesma transação.
* **`em.refresh(objeto)`:** Faz o contrário. Às vezes, as atualizações de Repeatable Read atrasam seu código e você **deseja** pegar as modificações mais recentes feitas por outro processo no banco para o ID que você tem no Cache. Chamando o `refresh()`, a JPA atropela as edições que você fez localmente, dispara um `SELECT` pro banco e injeta os dados reais do banco atualizando a sua instância!

---

Espero que este capítulo reforce bem os alicerces fundamentais por trás dos panos dos Mapeamentos. A partir daqui, as coisas ficarão mais fáceis. O uso consciente do `EntityManager`, dos estados e do cache ditarão o nível de perfomance que seus sistemas corporativos conseguirão atingir. Bons estudos e pratique com os laboratórios!
