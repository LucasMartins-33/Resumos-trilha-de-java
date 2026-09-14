# Capítulo 10: Putting it Together with Strings, Nested Loops and Debugging

Este capítulo consolida conceitos essenciais para a lógica de programação em Java, mergulhando no funcionamento da classe `String`, explorando estruturas de repetição (`while` e `for`) e as complexidades de loops aninhados e uso do debugger.

## 1. Trabalhando com Strings

Diferente de `int` ou `boolean`, **`String` não é um tipo primitivo**, mas sim um Objeto (uma Classe). Isso significa que variáveis do tipo String possuem diversos métodos utilitários embutidos. 

*   **Tamanho e Índices:** Uma String é basicamente uma sequência de caracteres. O primeiro caractere fica no índice `0` e o último no índice `length() - 1`. Espaços em branco também contam como caracteres.

### Principais Métodos da Classe String

*   **`.length()`:** Retorna o tamanho total da String.
*   **`.substring(inicio, fim)`:** Extrai um pedaço da String. 
    *   *Atenção:* O índice final é **exclusivo** (vai até ele, mas não o inclui). Ex: `substring(0, 2)` de "ABC" retorna "AB".
*   **`.charAt(index)`:** Retorna o caractere (`char`) exato que está naquela posição.
*   **`.indexOf("texto")`:** Pesquisa na String da esquerda para a direita e retorna em qual índice numérico aquele texto começa. Se não encontrar nada, retorna `-1`. 
    *   Você também pode passar um segundo argumento para dizer a partir de onde ele deve começar a procurar: `.indexOf("texto", 5)`.

> **⚠️ ARMADILHA MORTAL:** NUNCA compare Strings usando `==`. Como são objetos, o `==` compara o endereço de memória, não o texto. Sempre use:
> *   `str1.equals(str2)`: Verifica se os textos são idênticos.
> *   `str1.equalsIgnoreCase(str2)`: Verifica se são idênticos ignorando letras maiúsculas/minúsculas.

## 2. Estrutura de Repetição: `while`

O loop `while` (enquanto) executa um bloco de código repetidas vezes **enquanto uma condição for verdadeira**.

*   É fundamental alterar a variável de controle dentro do loop, caso contrário você criará um **Loop Infinito**.
*   A palavra-chave **`break`** pode ser usada para forçar a saída (término) do loop prematuramente se uma determinada condição for atendida.

**Sintaxe Básica:**
```java
int count = 0;
while (count < 100) {
    System.out.println("O contador está em: " + count);
    
    if (count == 50) {
        break; // Interrompe o loop no 50
    }
    
    count = count + 1; // Passo fundamental para o loop não ser infinito
}
```

## 3. Estrutura de Repetição: `for`

O loop `for` é uma versão condensada do `while`, ideal para quando você sabe exatamente quantas vezes precisa iterar (por exemplo, percorrer os caracteres de uma String de 0 até o tamanho dela).

Ele é dividido em 3 partes separadas por ponto e vírgula:
1.  **Inicialização:** Executada apenas uma vez no início (Ex: `int i = 0;`).
2.  **Condição:** Avaliada antes de cada iteração. Se `true`, o loop roda (Ex: `i < 100;`).
3.  **Incremento/Ação:** Executada ao final de cada iteração (Ex: `i++` ou `i = i - 1`).

**Percorrendo uma String de trás para frente:**
```java
String name = "Imtiaz";

// Começa do último índice (length - 1) e vai diminuindo (i--) até o 0
for (int i = name.length() - 1; i >= 0; i--) {
    System.out.println(name.charAt(i));
}
```

## 4. Nested Loops (Loops Aninhados) e Escalabilidade

Você pode colocar um loop dentro de outro loop. O loop mais interno irá rodar completamente **para cada iteração única** do loop externo.

**Exemplo:**
```java
for (int i = 0; i < 100; i++) {       // Roda 100 vezes
    for (int j = 0; j < 10; j++) {    // Roda 10 vezes
        // Este código será executado 1.000 vezes (100 * 10)
    }
}
```

> **💡 Dica de Ouro / Escalabilidade:** 
> Evite loops aninhados sempre que possível! Em pequenos conjuntos de dados isso não é problema, mas se você tiver um loop de 1.000 itens dentro de outro de 1.000 itens, o computador fará **1 milhão** de operações. Isso destrói a performance da aplicação (Problema de complexidade e escalabilidade). Tente sempre achar soluções mais otimizadas ou usar `break` para interromper o trabalho desnecessário cedo.

## 5. Debugger (Depurador)
O Debugger é uma ferramenta de raio-X do código. 
Você cria um **Breakpoint** (um ponto de parada clicando na margem esquerda do editor) e roda o programa em modo de depuração. 

O programa vai congelar naquela linha, permitindo que você:
*   Clique em **"Step Over"** para ir linha por linha em câmera lenta.
*   Veja os valores em tempo real de todas as variáveis mudando a cada passo.
*   Ache onde a lógica quebrou sem precisar espalhar centenas de `System.out.println()` pelo código.
