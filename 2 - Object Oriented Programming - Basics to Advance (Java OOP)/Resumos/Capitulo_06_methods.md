# Anotações de Estudo: Java Core - Capítulo 6 (Methods)

> [!NOTE]
> Este documento cobre toda a fundação sobre criação e uso de Métodos no Java. Preste bastante atenção ao tópico de **Pass by Value**, pois é uma pegadinha clássica em entrevistas de emprego e essencial para não causar bugs inesperados na alteração de estado dos objetos.

---

## 23. Methods Overview (Visão Geral)

Um método é um bloco de código que pode ser executado sempre que solicitado (chamado). Ele melhora a reutilização do código e o encapsulamento.
* **Nomenclatura (Clean Code)**: Um método deve indicar uma ação. Use sempre verbos em *camelCase*. Exemplo: `calculateFactorial`, `changeSecondaryMail`. Um código limpo poupa 50% do tempo de desenvolvimento futuro!
* **Assinatura do Método (Method Signature)**: No Java, a assinatura é composta **apenas pelo Nome do Método + Lista de Parâmetros**. O tipo de retorno e o modificador de acesso **NÃO** fazem parte da assinatura.

### Overloading (Sobrecarga de Métodos)
Como a assinatura engloba apenas Nome e Parâmetros, o Java permite que você crie dezenas de métodos com o **mesmo nome**, desde que as listas de parâmetros sejam diferentes (diferentes tipos ou quantidades).
* Exemplo: `System.out.println()` é um método altamente sobrecarregado (imprime string, int, double, boolean, etc.).

---

## 24. Parameter Passing Mechanism (Passagem de Parâmetros)

**A Regra de Ouro em Java**: Em Java, dados de tipo primitivo E de referência são **sempre passados por VALOR (Pass by Value)**. Nunca por referência.

> [!CAUTION]
> Muitas fontes na internet afirmam erroneamente que "objetos são passados por referência no Java". Isso é mentira e o professor foca bastante nisso.

O arquivo [PassByValueDemo.java](file:///C:/Users/lucas/OneDrive/Documentos/Cursos/learnit_java_core-master/learnit_java_core-master/src/com/itbulls/learnit/javacore/methods/PassByValueDemo.java) mostra exemplos práticos:

### Passando Primitivos (int, double)
O Java cria uma cópia da variável local na memória *Stack*. Se você alterar o número dentro do método, a variável fora do método **não** sofre alteração. (A única forma de refletir a mudança é o método dar um `return` do novo valor e a variável de fora receber).

### Passando Arrays ou Objetos
Quando você passa um Array, o Java **copia o valor da referência** (o "endereço da casa" na memória Heap). 
* Como você tem a cópia do endereço, se você mexer nos quartos da casa (ex: `array[1] = 200`), você estará alterando a casa original.
* PORÉM, se dentro do método você jogar o endereço fora e mandar apontar para o nada (ex: `array = null`), a variável original lá fora não é afetada, pois você só apagou a **sua cópia do endereço**, não a casa.

---

## 25. Recursive Methods (Métodos Recursivos)

Um método recursivo é aquele que chama a si mesmo.

* **Vantagens**: Muitas vezes, torna o design do código muito mais limpo do que se fossemos usar dezenas de `loops` (laços de repetição).
* **Desvantagens**: O grande risco de ocorrer um `StackOverflowError`. Cada vez que um método é chamado, o Java cria um "frame" empilhado na memória Stack. Se o método chamar a si mesmo infinitamente, a memória estoura.

> [!IMPORTANT]
> **Condição de Parada (Stop Condition)**: É obrigatório definir em que momento a recursão deve parar e os valores começarem a ser retornados.

Exemplo no projeto ([RecursiveMethodsDemo.java](file:///C:/Users/lucas/OneDrive/Documentos/Cursos/learnit_java_core-master/learnit_java_core-master/src/com/itbulls/learnit/javacore/methods/RecursiveMethodsDemo.java)):
```java
private static int calculateFactorial(int i) {
    if (i != 0) { // <-- Se não chegou no zero, continue chamando
        return i * calculateFactorial(i - 1);
    } else {
        return 1; // <-- Condição de Parada!
    }
}
```

---

## 26. Variable Length Arguments (Varargs)

Às vezes, você quer criar um método (como o método de Soma) mas não sabe quantos números o cliente quer somar. 
A solução são os **Varargs** usando as reticências `...`.

* Por trás dos panos, o Java transforma esse parâmetro em um Array (`[]`), então você pode iterar sobre ele usando um `for-each`.
* Exemplo do projeto: `private static int sum(int... ints) { ... }`

> [!WARNING]
> **Regra Obrigatória do Varargs**: Se o método tem vários parâmetros, o Vararg **tem obrigatoriamente que ser o último parâmetro** da lista. Se colocar no início (`void myMethod(int... numbers, String s)`), ocorrerá um erro de compilação, pois a JVM não saberá onde terminam os números e onde começa a string.

Até mesmo o método `main` padrão do Java pode ser modificado com Varargs:
```java
// O Java aceita e roda normalmente!
public static void main(String... args) {
    System.out.println("Funciona com Varargs também!");
}
```
