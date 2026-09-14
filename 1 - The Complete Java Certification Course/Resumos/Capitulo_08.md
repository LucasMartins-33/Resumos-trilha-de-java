# Capítulo 08: Understanding Object Orientation

Este capítulo mergulha no coração do Java: a **Orientação a Objetos (OO)**. Ele cobre a diferença teórica entre escrever o código e executá-lo, como a memória funciona e os pilares da OO (Herança, Abstração e Polimorfismo).

## 1. Classes vs. Objetos (Tempo de Escrita vs. Runtime)

*   **Classe (Class):** É a especificação, o "molde" ou a "planta" (blueprint). As classes contêm os campos (atributos) e métodos (comportamentos) que os futuros objetos terão. As classes em si **não fazem nada**; elas apenas ditam as regras.
*   **Objeto (Object / Instance):** São criados apenas quando o programa está **rodando (runtime)**. Você usa a classe como molde para instanciar (dar à luz) os objetos na memória.

**Sintaxe de Criação de Objeto:**
```java
// 1. Variável de Referência (Tipo Human)
// 2. Operador 'new' aloca memória para o objeto
// 3. Construtor Human() inicializa o objeto
Human tom = new Human(); 
```

## 2. Construtores e a palavra `this`

O **Construtor** é um método especial usado para inicializar o objeto quando usamos a palavra `new`. Ele **nunca tem retorno** (nem mesmo `void`) e **obrigatoriamente tem o mesmo nome da Classe**.

*   Se você não criar um construtor, o Java fornece um vazio e invisível por padrão.
*   A palavra `this` serve para o objeto referenciar a si mesmo, diferenciando o atributo da classe do argumento passado no método.

**Sintaxe de um Construtor Flexível:**
```java
public class Human {
    String name;
    int age;

    // Construtor
    public Human(String name, int age) {
        this.name = name; // this.name é o campo do objeto; name é o argumento recebido
        this.age = age;
    }
}
```

## 3. Entendendo a Memória: Stack vs. Heap e o Garbage Collector

*   **Stack (Pilha):** Onde ficam as chamadas de métodos (frames) e as **variáveis locais**. A execução flui empilhando e desempilhando métodos.
*   **Heap:** É um grande espaço de memória onde todos os **Objetos** (criados com `new`) vão morar.
*   **Variáveis de Referência:** Quando você digita `Car myCar = new Car();`, a variável `myCar` mora na *Stack*, mas o objeto carro real mora na *Heap*. A variável `myCar` apenas guarda o **endereço** do objeto (referência).
*   **Garbage Collection:** É o processo automático do Java que varre a Heap buscando objetos "órfãos" (que não têm mais nenhuma variável apontando para eles) e os destrói para liberar memória.

## 4. Herança (`extends`)

Permite organizar o código e evitar duplicação, fazendo com que uma classe "filha" (Subclass) herde todos os atributos e métodos de uma classe "pai" (Superclass/Base class).

*   O Java **não** permite herança múltipla (uma classe só pode ter UM pai).
*   Se o pai possui um construtor com argumentos, o filho obrigatoriamente precisa chamar o pai através da palavra-chave `super()`.

**Sintaxe de Herança:**
```java
public class Bird extends Animal { // Bird herda de Animal
    
    public Bird(int age, String gender, int weight) {
        // Chama o construtor da classe pai (Animal) para inicializar os atributos básicos
        super(age, gender, weight); 
    }
}
```

## 5. Interfaces (`implements`)

Interfaces são **contratos**. Uma interface apenas define *o que* deve ser feito, mas não *como* deve ser feito. Ela contém métodos **abstratos** (sem corpo). 
Qualquer classe que assinar esse contrato (usando `implements`) é **obrigada** a construir o corpo desses métodos.

*   Uma classe pode implementar **várias interfaces** ao mesmo tempo (diferente do `extends`).
*   O nome das interfaces costuma representar uma "habilidade" (ex: `Flyable`, `Eatable`, `Runnable`).

**Sintaxe de Interfaces:**
```java
public interface Flyable {
    // Método abstrato: não tem bloco de código { }
    void fly(); 
}

public class Sparrow extends Bird implements Flyable {
    // Obrigatoriamente precisamos implementar o método aqui:
    @Override
    public void fly() {
        System.out.println("Sparrow flying high!");
    }
}
```

## 6. Classes Abstratas (`abstract`)

São muito parecidas com classes normais, mas **não podem ser instanciadas** (você não pode dar `new` em uma classe abstrata). Elas servem unicamente como modelo para classes filhas.
Você as usa quando quer herdar métodos que já tenham um corpo (comportamento pronto), mas também quer obrigar os filhos a criarem seus próprios métodos (métodos abstratos).

**Sintaxe:**
```java
public abstract class Animal {
    // Método comum (já vem pronto para as filhas usarem)
    public void eat() {
        System.out.println("Eating...");
    }
    
    // Método abstrato (filhas serão OBRIGADAS a criarem esse comportamento)
    public abstract void move(); 
}
```

## 7. Polimorfismo e Visibilidade de Tipo

Polimorfismo significa "muitas formas". É o superpoder da orientação a objetos de tratar objetos específicos como se fossem genéricos, simplificando métodos.

*   A **Variável de Referência** define quais métodos você **pode ver e chamar** enquanto programa.
*   O **Objeto Instanciado** define **como** o método vai se comportar de fato quando rodar.

**Exemplo Mágico do Polimorfismo:**
```java
// O tipo da variável é genérico (Animal)
// Mas o objeto real que nasce é um Sparrow
Animal myPet = new Sparrow(); 

// Isso funciona e vai usar o "move()" escrito na classe Sparrow!
myPet.move(); 

// ERRO! myPet é do tipo Animal, e a classe Animal não tem o método 'fly()'.
// A IDE não deixa compilar, mesmo que o objeto real seja um pássaro.
// myPet.fly(); 
```

Você pode criar um método no sistema que movimenta animais de forma genérica, sem precisar saber se é peixe, pássaro ou cachorro:
```java
// Pode receber Sparrow, Fish, Dog... e cada um saberá se mover sozinho!
public static void moveAnimal(Animal animal) {
    animal.move(); 
}
```
