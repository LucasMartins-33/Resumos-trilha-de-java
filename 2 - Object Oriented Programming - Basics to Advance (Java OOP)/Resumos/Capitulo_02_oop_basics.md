# Anotações de Estudo: Java Core - Capítulo 2 (OOP Basics)

> [!NOTE]
> Este documento contém anotações detalhadas baseadas nas transcrições do curso e no código-fonte do projeto original (pasta `learnit_java_core-master`). Use estas anotações para consultar a sintaxe e os conceitos chave do Java sempre que precisar.

---

## 7. Object-Oriented Programming: Basics (Conceitos Básicos)

Embora não tenha havido transcrição para esta aula, aqui está o resumo essencial que você precisa saber sobre **Programação Orientada a Objetos (POO)**. POO é um paradigma de programação baseado no conceito de "objetos", que podem conter dados (campos/atributos) e código (métodos/comportamentos).

Os 4 pilares da POO no Java são:
1. **Encapsulamento**: Esconder os detalhes internos de como um objeto funciona (usando modificadores como `private`) e expor apenas o que é seguro e necessário (através de métodos públicos, como `getters` e `setters`).
2. **Herança**: Permite criar novas classes baseadas em classes existentes, promovendo o reuso de código (usamos a palavra-chave `extends`).
3. **Polimorfismo**: A capacidade de um objeto tomar muitas formas. Em Java, isso ocorre principalmente quando uma referência de um tipo pai (como uma interface) aponta para um objeto de um tipo filho.
4. **Abstração**: Focar apenas nas informações essenciais de um objeto e ignorar os detalhes irrelevantes. (Interfaces e classes abstratas são as ferramentas para isso).

---

## 8. Classes & Objects (Classes e Objetos)

Uma **Classe** é o molde (template) que define como será um objeto.
Um **Objeto** é uma instância gerada a partir desse molde, guardada na memória *Heap* do Java, que possui seu próprio estado.

### O que uma classe pode conter?
De acordo com o código [Cart.java](file:///C:/Users/lucas/OneDrive/Documentos/Cursos/learnit_java_core-master/learnit_java_core-master/src/com/itbulls/learnit/javacore/oop/classes/Cart.java), uma classe pode ter:

1. **Campos (Fields/Properties)**: Representam o estado do objeto. Podem ser estáticos ou não-estáticos.
2. **Blocos de Inicialização (Initialization Blocks)**:
   - **Não-estáticos**: Executados toda vez que um objeto é criado (antes do construtor). Úteis para inicializar variáveis comuns a todos os construtores.
   - **Estáticos**: Executados **apenas uma vez**, quando a classe é carregada pela JVM (útil para registrar *drivers* de banco de dados, por exemplo).
3. **Construtores (Constructors)**: Métodos especiais chamados com a palavra `new` para instanciar o objeto.
   - **Regra**: O nome é idêntico ao da classe, e **não** possui tipo de retorno (nem mesmo `void`).
   - Se você não criar nenhum construtor, o Java cria um construtor padrão vazio invisível. Se você criar um construtor com parâmetros, o construtor vazio deixa de existir automaticamente (a menos que você o escreva explicitamente).
4. **Métodos (Methods)**: Comportamentos que alteram ou consultam o estado do objeto. (Diferente de função, um método sempre pertence a um objeto em POO).
5. **Classes Aninhadas (Nested Classes)**: Classes declaradas dentro de outras classes.

> [!TIP]
> Use a palavra-chave `this` dentro de métodos ou construtores para referenciar a instância atual do objeto, resolvendo ambiguidades quando variáveis locais têm o mesmo nome que os atributos da classe.

### Getters, Setters e toString()
- **Getters/Setters**: Como os atributos são geralmente `private` (encapsulados), criamos métodos públicos para ler (get) e alterar (set) os dados. Isso nos permite adicionar regras de validação.
  - *Dica no Eclipse:* `Alt + Shift + S` -> "Generate Getters and Setters".
- **toString()**: Um método que retorna uma representação em texto (`String`) legível do seu objeto, em vez de imprimir o endereço de memória. Ele é chamado automaticamente ao dar um `System.out.println(objeto)`.

### Exemplo de Sintaxe (Baseado no projeto original)
```java
public class Cart {
    // 1. Campos
    private int id;
    private static int cartCounter;
    private Product[] products;

    // 2. Bloco Estático (Roda 1 vez)
    static {
        System.out.println("Cart.class is uploaded into JVM");
    }

    // 3. Bloco de Inicialização (Roda a cada 'new Cart()')
    {
        cartCounter++;
    }

    // 4. Construtores
    public Cart() { } // Construtor Vazio
    
    public Cart(int id) {
        this.id = id; // Uso do this
    }

    // 5. Getter defensivo (Retorna uma cópia do array para não quebrarem o estado interno)
    public Product[] getProducts() {
        return Arrays.copyOf(products, products.length);
    }
    
    // 6. Setter com validação
    public void setId(int id) {
        if (id < 0) return; // Validação: Impede ID negativo
        this.id = id;
    }
}
```

---

## 9. Tipos de Classes (Different types of Classes)

O Java permite criarmos diferentes "estilos" de classes dependendo do objetivo arquitetural:

1. **Concrete Classes (Classes Concretas)**: São classes comuns, onde **todos** os métodos possuem implementação (corpo). Você pode criar objetos diretos com o `new`.
2. **Nested Static Classes (Classes Estáticas Aninhadas)**: Declaradas dentro de outra classe usando a palavra `static`. Ex: `public static class Tax {}`. Elas não dependem da classe externa para existir. Instancia-se assim: `new Cart.Tax()`.
3. **Inner Classes (Classes Internas)**: Declaradas dentro de outra classe **sem** a palavra `static`. Ex: `public class Discount {}`. Estão amarradas à instância externa. Para criá-las, precisa antes ter o objeto pai:
   ```java
   Cart meuCarrinho = new Cart();
   Cart.Discount meuDesconto = meuCarrinho.new Discount();
   ```
4. **Final Classes**: Classes com o modificador `final`. **Não podem ser herdadas** (`extends` é proibido). Ex: `String` é uma classe `final` no Java.
5. **POJO (Plain Old Java Object)**: Um "Objeto Java Comum". É um termo para descrever classes simples que só tem atributos, getters/setters e um construtor padrão. Não herdam de frameworks complicados nem usam anotações complexas.
6. **Abstract Classes (Classes Abstratas)**:
   - Usadas para agrupar código comum numa hierarquia (ex: Classe abstrata `Product` servindo de base para `MasterProduct` e `VariantProduct`).
   - **Não podem ser instanciadas diretamente** (`new Product()` dá erro).
   - Podem conter **Métodos Abstratos** (sem corpo, apenas assinatura, terminando em `;`), obrigando as classes filhas a implementarem esse método.
7. **Anonymous Classes (Classes Anônimas)**: Definidas e instanciadas no mesmo momento, sem receber um nome formal. Muito usadas antes das expressões Lambda para implementar interfaces "on the fly" (em tempo real).

---

## 10. Interfaces

Uma Interface é uma **abstração** pura que define um **contrato**. Ela dita *o que* deve ser feito, mas não *como* deve ser feito.

### O que uma interface contém?
De acordo com [PaymentProcessor.java](file:///C:/Users/lucas/OneDrive/Documentos/Cursos/learnit_java_core-master/learnit_java_core-master/src/com/itbulls/learnit/javacore/oop/interfaces/PaymentProcessor.java):
1. **Métodos abstratos**: Declarar um método apenas com assinatura. Por baixo dos panos, eles já são `public abstract`.
2. **Constantes**: Todas as variáveis em interfaces são automaticamente `public static final`.
3. **Default Methods (Java 8+)**: Métodos *com* corpo. Foram adicionados para permitir que interfaces evoluam no futuro sem quebrar o código de quem já as implementava. Usam a palavra `default`.
4. **Static Methods**: Pertencem à própria interface e podem ser chamados sem instanciar nada (`PaymentProcessor.someStaticMethod()`).

> [!WARNING]
> Uma classe no Java só pode **estender** (`extends`) UMA outra classe. Porém, ela pode **implementar** (`implements`) MÚLTIPLAS interfaces. Se a classe implementar duas interfaces que tenham *default methods* com assinaturas idênticas, ocorrerá um erro de compilação, e você será obrigado a sobrescrever o método para decidir o que fazer.

### Diferença entre Type (Tipo) e Object (Objeto)
- O **Tipo** define o conjunto de comportamentos e características. Uma classe é um tipo, uma interface é um tipo.
- O **Objeto** é a implementação na memória.
- No princípio de polimorfismo, programamos orientados a "Tipos" (Interfaces), e não a Classes:
  ```java
  // Boa prática: A variável é do Tipo da Interface, mas o Objeto é da classe Concreta
  PaymentProcessor paypal = new PayPalPaymentProcessor();
  ```

### Exemplo de Sintaxe de Interface
```java
public interface PaymentProcessor {
    
    // Constante (public static final estão implícitos)
    int RETRY_ATTEMPTS = 5;
    
    // Método abstrato (public abstract estão implícitos)
    void processPayment(PaymentData payment);
    
    // Método Default (com corpo)
    default void someDefaultMethod() {
        System.out.println("This is the default method");
    }
}

// Classe implementando a interface
public class PayPalPaymentProcessor implements PaymentProcessor {
    @Override
    public void processPayment(PaymentData payment) {
        // Lógica específica do PayPal
    }
}
```
