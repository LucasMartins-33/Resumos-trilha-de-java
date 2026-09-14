# Questões Teóricas - Capítulo 04 (Variáveis, Fluxo de Controle e Loops)

**1. Tipos Primitivos:** Liste os 8 tipos de dados primitivos disponíveis na linguagem Java e descreva resumidamente a principal diferença entre os tipos numéricos inteiros (como `int` e `long`) e os de ponto flutuante (`float` e `double`).
<details>
<summary>👀 Ver Resposta</summary>

Os 8 tipos são: `byte`, `short`, `int`, `long` (inteiros), `float`, `double` (ponto flutuante), `boolean` (verdadeiro/falso) e `char` (caractere único). A diferença é que inteiros armazenam números exatos sem casas decimais, enquanto ponto flutuante permite frações (casas decimais).
</details>

**2. Declaração vs Inicialização:** Qual é a diferença exata entre "declarar" uma variável e "inicializar" uma variável? O que acontece no compilador do Java se tentarmos ler o valor de uma variável local que foi declarada, mas nunca inicializada?
<details>
<summary>👀 Ver Resposta</summary>

Declarar é reservar o espaço e dar um nome e tipo (ex: `int x;`). Inicializar é atribuir o primeiro valor a ela (ex: `x = 10;`). Se tentarmos ler/usar uma variável local não inicializada, o compilador do Java lançará um erro em tempo de compilação (Variable might not have been initialized) e o programa não rodará.
</details>

**3. Escopo de Variáveis:** Explique o conceito de "escopo" (scope) em Java. Se uma variável for declarada dentro de um bloco condicional `if { ... }`, ela poderá ser acessada fora desse bloco? Por quê?
<details>
<summary>👀 Ver Resposta</summary>

Escopo é a área de visibilidade de uma variável no código, delimitada pelas chaves `{}`. Se for declarada dentro do `if`, ela só existe lá dentro (escopo de bloco). Ao sair das chaves, ela é destruída pela memória, portanto não pode ser acessada de fora, gerando erro de compilação.
</details>

**4. While vs Do-While:** Qual é a principal diferença de comportamento entre um loop `while` e um loop `do-while`? Dê um exemplo prático de quando seria obrigatório (ou mais recomendado) o uso do `do-while`.
<details>
<summary>👀 Ver Resposta</summary>

O `while` testa a condição *antes* de executar o bloco (pode rodar 0 vezes). O `do-while` testa a condição *depois*, garantindo que o bloco rode **pelo menos 1 vez**. É recomendado para menus interativos, onde você precisa exibir as opções na tela ao menos uma vez antes de checar se o usuário quer sair.
</details>

**5. Estrutura For:** A declaração de um loop `for` clássico em Java é composta por três partes separadas por ponto-e-vírgula (ex: `for (A; B; C)`). Descreva o que cada uma dessas três partes faz e em que momento elas são executadas durante o ciclo de vida do loop.
<details>
<summary>👀 Ver Resposta</summary>

`A` é a inicialização (ex: `int i = 0`), roda apenas 1 vez no início. `B` é a condição (ex: `i < 10`), avaliada *antes* de cada iteração para decidir se o loop continua. `C` é o incremento/atualização (ex: `i++`), roda sempre ao final de cada iteração do loop.
</details>

**6. Switch vs If/Else:** Em quais cenários arquiteturais ou lógicos é mais recomendado utilizar a estrutura `switch` ao invés de uma longa cadeia de `if / else if`? Há alguma limitação sobre os tipos de dados que podemos colocar na cláusula `switch(X)` no Java?
<details>
<summary>👀 Ver Resposta</summary>

O `switch` é recomendado quando você está avaliando o valor exato de uma única variável contra múltiplas opções diretas, deixando o código mais limpo. Limitações: o `switch` tradicional do Java não aceita operadores relacionais (`>`, `<`). Ele aceita os tipos primitivos convertíveis pra int (`byte`, `short`, `char`, `int`), suas classes Wrapper, `Enums` e `String` (a partir do Java 7). Não aceita `boolean`, `float` ou `double`.
</details>

**7. Break e Continue:** Explique com suas palavras a diferença entre as palavras-chave `break` e `continue` quando utilizadas dentro de uma estrutura de repetição. 
<details>
<summary>👀 Ver Resposta</summary>

`break` "quebra" e encerra totalmente o loop imediatamente, pulando para a linha fora dele. `continue` ignora apenas o restante do código da iteração atual e "pula" para o início do próximo ciclo/iteração do loop.
</details>

**8. Operadores Lógicos de Curto-Circuito (Short-Circuit):** Dado um `if (condicao1 && condicao2)`, explique como o operador `&&` avalia as condições. Se a `condicao1` for falsa, o Java chegará a avaliar a `condicao2`? Por quê?
<details>
<summary>👀 Ver Resposta</summary>

Não chegará a avaliar. Por ser um "E" (AND), ambas precisam ser verdadeiras. Se o Java percebe que a `condicao1` é Falsa, o resultado final da expressão obrigatoriamente será Falso, então ele entra em curto-circuito (pula) e ignora a `condicao2` para economizar processamento e evitar NullPointerExceptions.
</details>

**9. Casting de Variáveis:** O que significa o processo de "Casting" em Java? Qual é a diferença entre um casting implícito (widening) e um casting explícito (narrowing) ao trabalhar com variáveis primitivas?
<details>
<summary>👀 Ver Resposta</summary>

Casting é a conversão de um tipo de dado em outro. Implícito (widening) ocorre automaticamente quando passamos de um tipo menor para um maior (ex: `int` para `double`), pois não há perda de dados. Explícito (narrowing) exige que forçemos a conversão colocando o tipo entre parênteses (ex: `(int) 2.5`), pois estamos indo de um tipo maior para um menor e correndo o risco de perder dados (como perder as casas decimais).
</details>

**10. Loops Infinitos:** O que causa um "loop infinito" (infinite loop) em um programa Java? Escreva (em pseudo-código ou texto) um pequeno exemplo de um `while` que rodaria infinitamente por erro do programador.
<details>
<summary>👀 Ver Resposta</summary>

Ocorre quando a condição de parada (término) do loop nunca se torna falsa. Exemplo: `int x = 0; while(x < 5) { System.out.println("Oi"); }` — o `x` nunca foi atualizado/incrementado para chegar em 5, logo ficará travado imprimindo "Oi" para sempre.
</details>




