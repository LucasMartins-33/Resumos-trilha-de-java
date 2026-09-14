# Capítulo 06: Understanding Methods

Este capítulo é um dos mais fundamentais da linguagem Java. Ele introduz o conceito de **Métodos** (Methods), modificadores de acesso, e a diferença inicial entre contextos estáticos e de instância (objetos).

## 1. O que são Métodos?

Métodos são blocos nomeados de instruções. Em vez de escrever todo o código de uma vez no `main`, nós o dividimos em pequenos blocos com responsabilidades específicas que podem ser "chamados" (invoked/called) repetidamente em qualquer parte do programa.

*   `System.out.println("Texto")` é um exemplo de método integrado do Java.
*   **Argumentos/Parâmetros:** São os dados que passamos para o método quando o chamamos (ex: `"Texto"` acima).

## 2. Definindo e Chamando seus Próprios Métodos

Um método é dividido em duas partes principais: a **Assinatura do Método** (Method Signature) e o **Corpo do Método** (Method Body).

### 2.1 Métodos Sem Retorno (`void`) e Sem Argumentos
A palavra `void` significa "vazio". Ela indica que o método faz o seu trabalho (como imprimir algo na tela), mas não retorna nenhum valor para a variável que o chamou.

**Sintaxe Básica:**
```java
public class LearningMethods {
    
    public static void main(String[] args) {
        // Chamando o método:
        printSomeJunk(); 
    }

    // Assinatura do método
    public static void printSomeJunk() {
        // Corpo do método
        System.out.println("Isso é apenas um texto impresso pelo método!");
    }
}
```

### 2.2 Métodos com Argumentos
Podemos configurar os métodos para receber dados externos (parâmetros) especificando o tipo e um nome de variável dentro dos parênteses `()`.

**Sintaxe:**
```java
public static void printMyName(String name) {
    System.out.println("Hello, my name is " + name);
}

// Chamando:
// printMyName("Lucas");
```
> **Nota de Erro:** Se você passar um número (`int`) para um método que exige uma `String`, o código nem irá compilar (Type Mismatch). 

### 2.3 Métodos com Retorno
Se você quiser que o método faça um cálculo e **devolva** o resultado para você salvar em uma variável, você deve remover o `void` e colocar o tipo do dado esperado, além de usar a palavra-chave `return`.

**Sintaxe:**
```java
public static int add10(int someArgument) {
    int result = someArgument + 10;
    return result; // Obrigatoriamente precisa retornar um int
}

// Chamando e capturando o retorno no método main:
// int total = add10(99); 
// System.out.println(total); // Imprime 109
```

## 3. Visibilidade (Modificadores de Acesso)

A palavra `public` não está ali à toa. Ela dita de onde o seu método (ou classe) pode ser chamado/enxergado.

*   `public`: O método é visível e pode ser invocado por **qualquer classe** em qualquer pacote.
*   `private`: O método só pode ser invocado e visto **dentro da própria classe** onde foi criado. Se tentar chamar de fora, o Eclipse dará um erro de compilação.
*   **Sem modificador (Default/Package-Private):** Se você apagar a palavra `public` e não colocar nada, a classe/método só será visível para arquivos que estiverem **dentro do mesmo pacote** (`package`).

**Exemplo Prático - Chamando métodos de outra classe:**
```java
// Se MyUtils for public e estiver em outro pacote, precisamos importar!
// import someotherpackage.MyUtils;

public static void main(String[] args) {
    // Chamando um método público de outra classe
    MyUtils.sum2Numbers(10, 23); 
}
```

## 4. O Mistério do `static` (Static vs Instância)

Até agora, todos os métodos usaram a palavra `static`. 

### Métodos `static` (Da Classe)
Se um método é `static`, ele pertence à **Classe** diretamente. Você não precisa criar uma cópia da classe na memória. Basta digitar o nome da classe seguido do método:
```java
MyUtils.printSomeJunk(); // Chamada estática
```

### Métodos Não-Estáticos / de Instância (Objetos)
Se você remover a palavra `static` da assinatura do método, ele passa a pertencer a uma **instância** (um objeto criado a partir da classe). Você não pode chamá-lo diretamente pelo nome da classe.

Para usar um método não-estático, é obrigatório criar uma variável do tipo daquela classe e inicializá-la com a palavra `new`:

**Sintaxe (O Início da Orientação a Objetos):**
```java
// MyUtils.add10(5); -> DARIA ERRO, pois add10 não é estático!

// 1. Criamos a variável (o Objeto)
MyUtils myVar = new MyUtils(); 

// 2. Chamamos o método a partir do objeto instanciado
int res = myVar.add10(5); 
```

## 5. Dissecando o Método `main`
Agora que sabemos todos os conceitos, fica fácil entender por que o método inicial do Java é escrito assim:
`public static void main(String[] args)`

*   `public`: Precisa ser acessível pela Máquina Virtual do Java (JVM) de fora da aplicação para iniciá-la.
*   `static`: O Java precisa rodar esse método sem precisar instanciar/criar um objeto da sua classe inicial.
*   `void`: O programa em si (esse método inicial) não devolve nenhum dado para o computador após terminar.
*   `main`: O nome padrão definido pelos criadores do Java.
*   `String[] args`: Aceita um Array de Strings como argumento. Isso permite que você passe dados para o seu programa através do terminal/linha de comando na hora de executá-lo.

---
> **💡 Boas Práticas Atuais:** 
> O curso ensina o conceito fundamental de `static`. Em códigos Java modernos, você evitará usar `static` para regras de negócio e lógica pesada, priorizando a **Orientação a Objetos** (criar o `new Objeto()`). Métodos `static` são frequentemente reservados para métodos utilitários, constantes ou funções de ajuda puras (como a classe nativa `Math` do Java).
