📘 Capítulo 6: Métodos e Manipulação de Dados

**O Cenário:**
Você está montando uma biblioteca de utilitários matemáticos e validações.

```java
public class Utilitarios {
    // Métodos serão adicionados aqui
}
```

Sua missão é exercitar sobrecarga, pass-by-value, recursão e varargs:

🟢 Atividade 6.1: Criando Métodos Simples
1. Crie o método estático `public static int somar(int a, int b)` que retorna a soma.
2. Invoque-o no método `main` de outra classe sem precisar instanciar `Utilitarios`.

🟢 Atividade 6.2: Sobrecarga (Overloading) Básica
1. Crie uma sobrecarga do método `somar` para que ele receba e retorne tipos `double`.
2. Crie uma terceira sobrecarga que receba três parâmetros `int`.

🟡 Atividade 6.3: Pass-by-value com Primitivos
1. Crie um método `public static void dobrarValor(int numero)` que internamente faz `numero = numero * 2;`.
2. No `main`, declare `int x = 10;`, chame `dobrarValor(x);` e imprima `x`.
3. Comente no código por que `x` continuou sendo 10.

🟡 Atividade 6.4: Pass-by-value com Objetos (Estado)
1. Crie uma classe auxiliar `Caixa` com um atributo `int valor;`.
2. Crie o método `public static void dobrarCaixa(Caixa c)` que altera o estado: `c.valor = c.valor * 2;`.
3. Teste no `main` e comente por que dessa vez o valor externo foi modificado.

🟠 Atividade 6.5: Pass-by-value com Objetos (Reatribuição)
1. Crie o método `public static void trocarCaixa(Caixa c)` que internamente faz `c = new Caixa(); c.valor = 999;`.
2. No `main`, passe sua caixa para esse método.
3. Imprima o valor e explique por que a reatribuição da referência dentro do método não vazou para o `main`.

🟠 Atividade 6.6: Varargs (Múltiplos Argumentos)
1. Crie o método `public static int somarTodos(int... numeros)`.
2. Implemente um laço *foreach* iterando sobre o array `numeros` para somar todos.
3. Teste chamando `somarTodos(1, 2, 3, 4, 5)`.

🔴 Atividade 6.7: Regras do Varargs
1. Tente alterar o método varargs para: `public static void relatorio(int... notas, String nomeAluno)`.
2. Veja o erro do compilador.
3. Corrija o erro aplicando a regra de que o varargs deve ser o último parâmetro.

🔴 Atividade 6.8: Recursão Clássica (Fatorial)
1. Crie o método `public static int fatorial(int n)`.
2. Defina o caso base: se `n == 0` ou `n == 1`, retorne 1.
3. Chame a recursão no passo recursivo: `return n * fatorial(n - 1);`.

🔴 Atividade 6.9: O Perigo da Recursão (StackOverflow)
1. Crie um método `public static void recursaoInfinita(int n)` que apenas imprima `n` e chame a si mesmo com `n+1`.
2. Remova qualquer caso base.
3. Execute e observe o `StackOverflowError` no console.

🔴 Atividade 6.10: Retornos Prematuros (Guard Clauses)
1. Crie um método `public static void processarDocumento(String doc)`.
2. Em vez de usar um bloco `if (doc != null) { ... }` gigante, utilize um `return;` imediato (Guard Clause) se o documento for nulo ou vazio.
3. Simule a lógica principal abaixo da validação.
