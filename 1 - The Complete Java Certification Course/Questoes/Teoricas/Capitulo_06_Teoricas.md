# Questões Teóricas - Capítulo 06 (Understanding Methods)

**1. A Anatomia do Método:** O que é a assinatura de um método (Method Signature) e quais partes compõem a estrutura básica de um método em Java?
> [!faq]- 👀 Ver Resposta
> A assinatura e a estrutura são formadas pelo Modificador de Acesso (ex: `public`), o comportamento de instância (ex: `static` se for classe), o Tipo de Retorno (ex: `void` ou `int`), o Nome do Método e a lista de Argumentos/Parâmetros nos parênteses. O "corpo" do método fica dentro das chaves `{ }`.

**2. Retorno vs Void:** Qual a diferença fundamental entre um método marcado como `void` e um método que retorna um valor (como `int` ou `String`)? O que o compilador faz se tentarmos "capturar" o resultado de um método `void` numa variável?
> [!faq]- 👀 Ver Resposta
> `void` significa vazio; o método executa uma tarefa, mas não devolve nenhum dado útil que possa ser armazenado. Se tentarmos colocar seu resultado numa variável (ex: `String resultado = meuMetodoVoid()`), o Java dará erro de compilação, pois não há valor a ser salvo. Já os outros tipos exigem o comando `return` e devolvem um dado.

**3. O Retorno Obrigatório:** O que acontece se um método for declarado com a assinatura `public static int calcular()` e esquecermos de usar a palavra-chave `return` no corpo dele, ou tentarmos usar `return "Erro"`;?
> [!faq]- 👀 Ver Resposta
> O código não compilará. O Java é estritamente tipado. Se você prometeu retornar um `int`, o compilador o forçará a ter pelo menos um `return numeroInteiro;` válido em todos os caminhos possíveis de execução. Passar uma String quando se pediu um int gera um erro de "Type Mismatch".

**4. Visibilidade:** Qual a diferença prática entre os modificadores de acesso `public` e `private` aplicados a um método?
> [!faq]- 👀 Ver Resposta
> Um método `public` pode ser chamado de qualquer lugar, mesmo de arquivos de outras pastas (pacotes). Um método `private` é restrito e só pode ser visto e invocado de dentro das próprias chaves do arquivo (classe) onde ele foi criado. Útil para "esconder" lógica interna sensível.

**5. Visibilidade Default (Package-Private):** Se você apagar a palavra `public` e não colocar nenhum modificador antes de `static void meuMetodo()`, quem terá acesso para invocar esse método?
> [!faq]- 👀 Ver Resposta
> Ele terá o acesso conhecido como "Package-Private" ou Default. Apenas as classes que estiverem exatamente no mesmo diretório (pacote / package) conseguirão enxergá-lo. Classes em outros pacotes não terão acesso.

**6. Static vs Não-Estático:** Qual é a diferença na forma de invocar (chamar) um método declarado como `static` em comparação a um método de instância (que não possui a palavra `static`)?
> [!faq]- 👀 Ver Resposta
> Um método `static` pertence à Classe, e não precisa ser instanciado, podendo ser chamado pelo nome do arquivo (ex: `Math.pow(2,3)`). Um método não-estático pertence ao Objeto, exigindo que você o construa na memória usando a palavra 
ew` (ex: `Carro c = new Carro(); c.acelerar()`).

**7. A Palavra-Chave 'new':** Por que precisamos da palavra-chave 
ew` ao chamar um método não-estático de uma classe como a `Utils`?
> [!faq]- 👀 Ver Resposta
> O 
ew` aloca um novo espaço na memória RAM para dar "vida" a uma instância única daquela classe (um objeto). Como métodos não-estáticos pertencem ao estado do objeto, eles só passam a existir e a poder rodar depois que esse molde ganha uma manifestação física na memória através do 
ew`.

**8. O Ponto de Entrada 'public':** Dissecando a declaração `public static void main(String[] args)`, explique por que o método `main` obrigatoriamente precisa ter o modificador `public`.
> [!faq]- 👀 Ver Resposta
> Ele precisa ser `public` para que a JVM (Máquina Virtual do Java), que é um programa externo rodando no seu Sistema Operacional, consiga enxergar e invocar o método inicial para dar a largada na execução do seu programa.

**9. O Ponto de Entrada 'static':** Ainda no método `main`, por que ele precisa obrigatoriamente conter a palavra `static`?
> [!faq]- 👀 Ver Resposta
> A JVM quer rodar o seu programa diretamente através da classe, sem ter que descobrir como construir (instanciar com 
ew`) o objeto principal primeiro. O `static` permite que o Java chame o `main` imediatamente apenas apontando para a classe base, poupando a máquina de tentar instanciar algo que ela nem conhece direito.

**10. Boas Práticas Estáticas:** É uma boa prática, pensando em sistemas grandes com Orientação a Objetos moderna, declarar todos os métodos de todas as classes como `static`? Por quê?
> [!faq]- 👀 Ver Resposta
> Não, é uma péssima prática. Ao colocar tudo como `static`, você perde as grandes vantagens da Orientação a Objetos, como Herança, Polimorfismo e a capacidade de ter vários objetos com "memórias" diferentes rodando ao mesmo tempo. O `static` costuma ser reservado para utilitários puros ou constantes.




