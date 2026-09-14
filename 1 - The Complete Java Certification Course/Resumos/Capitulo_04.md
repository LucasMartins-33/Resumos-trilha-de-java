# Capítulo 4: The Very Basics of Java

Este capítulo aborda os fundamentos absolutos da linguagem Java, desde a criação da sua primeira classe até estruturas de controle de fluxo e tipos de dados.

## 1. Estrutura Básica e Método Principal (`main`)
Toda aplicação Java precisa de uma classe e um ponto de entrada chamado método `main`. A execução do programa ocorre linha por linha, de cima para baixo, executando cada instrução sequencialmente.

**Sintaxe Básica:**
```java
public class Variables { // Nome da classe sempre com a primeira letra maiúscula (PascalCase)
    public static void main(String[] args) {
        // Todo o código de execução do programa começa aqui dentro
    }
}
```

## 2. Variáveis e Tipos de Dados

Variáveis são espaços na memória para armazenar dados temporariamente. A linguagem Java é **fortemente tipada**, o que significa que você deve sempre declarar o tipo de dado que a variável irá armazenar antes de usá-la.

*   **Declaração:** Informar o tipo e o nome da variável.
*   **Atribuição:** Dar um valor usando o operador `=` (Diferente da matemática, onde `=` é igualdade, na programação um único `=` significa atribuição).

### Tipos Primitivos (Built-in)
São os blocos de construção básicos integrados na linguagem:

*   `byte`: Números inteiros muito pequenos (suporta de -128 a 127).
*   `short`: Números inteiros curtos (suporta de -32.768 a 32.767).
*   `int`: Números inteiros padrão (32 bits). É o mais utilizado no dia a dia.
*   `long`: Números inteiros muito grandes (64 bits). Necessita do sufixo `L`.
*   `double`: Números com casas decimais (ponto flutuante).
*   `boolean`: Valores lógicos de tomada de decisão (`true` ou `false`).
*   `char`: Apenas um único caractere envolto por aspas simples `' '`.

**Sintaxe de Variáveis Primitivas:**
```java
int x = 34;
x = 23; // Reatribuição (o valor pode variar e ser alterado no meio do caminho)

long bigNumber = 100000000000L; // Uso obrigatório do 'L' no final
byte reallySmallNumber = 127; 
double decimalVariable = 394.003;
boolean isHungry = true; // pode ser apenas true ou false
char letter = 'T'; // aspas simples!
```

### O Tipo `String`
O tipo `String` é utilizado para armazenar textos (palavras, frases). Não é um tipo primitivo, mas sim uma Classe, porém é de uso cotidiano.
*   Utiliza sempre **aspas duplas** `" "`.
*   Suporta o operador `+` para **concatenação** (juntar textos).

**Sintaxe de Strings:**
```java
String words = "This is a sentence.";
String moreWords = words + " And this is appended."; // Resultado: "This is a sentence. And this is appended."
```

> **💡 Atualização de Sintaxe (Java 10+):** 
> Embora o Java seja fortemente tipado, em versões mais modernas foi introduzida a palavra-chave `var` para **inferência de tipo em variáveis locais**. Isso quer dizer que, se você inicializar a variável no mesmo momento que a declara, o Java consegue "adivinhar" o tipo para você sem que precise escrever explicitamente:
> ```java
> var age = 25; // O Java deduz automaticamente que é um int
> var name = "Lucas"; // O Java deduz automaticamente que é uma String
> ```

## 3. Trabalhando com Arrays

Arrays (vetores/matrizes) são maneiras de armazenar **múltiplos elementos do mesmo tipo** em uma única variável.
*   **Tamanho Fixo:** Ao inicializar um array, seu tamanho não pode mais ser alterado. Se precisar de mais espaço, tem que criar um novo.
*   **Índice baseado em Zero:** O primeiro "slot" do array é o índice `0`, e o último é o `tamanho - 1`.
*   Acessar um índice que não existe (ex: tentar acessar o slot 100 de um array que tem tamanho 100, cujo último índice é 99) gera o erro `ArrayIndexOutOfBoundsException`.
*   Slots criados e não preenchidos ganham valores padrão automaticamente (ex: `0` para números int/double, `null` para Strings).

**Sintaxe de Arrays:**
```java
// Forma 1: Declarar o tamanho e preencher depois
int[] values = new int[100]; // Inicializa um Array com 100 slots vazios (índices 0 a 99)
values[0] = 1000; // Colocando um dado no primeiro slot
values[99] = 5432; // Colocando um dado no último slot

// Forma 2: Iniciar já com os valores preenchidos (o tamanho e os slots são inferidos)
String[] words = {"My", "name", "is"}; // Cria um array de tamanho 3 com os índices 0, 1 e 2
System.out.println(words[2]); // Imprime a palavra "is"
```

## 4. Controle de Fluxo: `if`, `else if`, `else`

Utilizados para pular ou executar certos blocos de código condicionalmente, quebrando a sequência padrão de "cima para baixo". A execução depende de testes lógicos.

*   **Operadores de Comparação:** `==` (igualdade), `!=` (diferente), `<` (menor), `>` (maior), `<=` (menor ou igual), `>=` (maior ou igual).
*   **Operador de Negação (Bang/NOT):** `!` inverte o valor lógico de uma expressão (true vira false, e false vira true). Ex: `!true` é avaliado como `false`.

**Sintaxe `if-else`:**
```java
int currentTemp = 60;
int favoriteTemp = 75;

if (currentTemp == favoriteTemp) { // Checa se é exatamente igual
    System.out.println("It's perfect out!");
} else if (currentTemp < favoriteTemp - 30) {
    System.out.println("It's Pretty Darn Cold ...");
} else if (currentTemp > favoriteTemp + 10) {
    System.out.println("It's hot out.");
} else {
    // Bloco padrão: Executa se nenhuma das condições IF ou ELSE IF acima for verdadeira
    System.out.println("It's a beautiful day..."); 
}

// Usando a Negação (!)
boolean isHungry = false;
if (!isHungry) { 
    System.out.println("Não estou com fome"); 
}
```

## 5. Controle de Fluxo: `switch`

O `switch` é uma alternativa de estrutura para quando se tem muitas decisões que dependem do valor exato de uma mesma variável. É útil para substituir longas cadeias de `else if`.
*   A palavra `break` é essencial para "quebrar/sair" do `switch` depois que uma condição (`case`) é satisfeita.
*   O caso `default` age de maneira similar ao `else` final; ele é acionado se nenhum dos casos acima for satisfeito.

**Sintaxe do `switch` Clássico (Demonstrado no Curso):**
```java
int month = 2;
String monthString;

switch (month) {
    case 1:
        monthString = "January";
        break; // Extremamente importante para impedir a execução dos casos abaixo
    case 2:
        monthString = "February";
        break;
    default:
        monthString = "Unknown Month";
        break; // Boa prática colocar no default também
}
```

> **💡 Atualização de Sintaxe (Java 14+):** 
> O Java evoluiu bastante o `switch` adicionando as **Switch Expressions**. Elas são mais curtas, seguras, dispensam a escrita repetitiva da palavra `break` (evitando bugs) e ainda podem retornar um valor diretamente para uma variável:
> ```java
> int month = 2;
> String monthString = switch (month) {
>     case 1 -> "January";
>     case 2 -> "February";
>     default -> "Unknown Month";
> };
> ```
