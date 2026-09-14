# Questões Teóricas: Capítulo 6 - Métodos em Java

1. **Assinatura de Método:** Quais elementos de um método compõem a sua verdadeira "assinatura" perante o compilador do Java? (Aviso: O tipo de retorno faz parte da assinatura?)
<details>
<summary>👀 Ver Resposta</summary>

Em Java, a assinatura de um método é composta estritamente por: **o Nome do Método** e a **Lista de Tipos de Parâmetros** (incluindo sua quantidade e ordem exata). O tipo de retorno, os modificadores de acesso (`public`, `private`), o modificador `static` e as exceções declaradas na cláusula `throws` **NÃO fazem parte da assinatura do método**. Portanto, dois métodos na mesma classe não podem ter o mesmo nome e mesmos tipos de parâmetros apenas alterando o retorno, pois isso gera erro de compilação.
</details>

2. **Sobrecarga (Overloading):** Para sobrecarregar um método validamente, o que exatamente precisa ser alterado em sua assinatura?
<details>
<summary>👀 Ver Resposta</summary>

Para realizar uma sobrecarga válida de métodos (*overload*), o método deve manter o **mesmo nome**, mas **obrigatoriamente alterar a lista de parâmetros**, seja pela **quantidade** de parâmetros, pelos **tipos** de dados dos parâmetros ou pela **ordem** dos tipos declarados. Alterar apenas o tipo de retorno ou modificador de acesso sem mudar a lista de parâmetros é inválido e causará erro de método duplicado.
</details>

3. **Pass-by-value:** Explique o mecanismo de "passagem por valor" utilizado no Java ao passar argumentos para um método.
<details>
<summary>👀 Ver Resposta</summary>

O Java utiliza **estritamente e exclusivamente a passagem por valor** (*pass-by-value*). Isso significa que, ao invocar um método, o Java cria uma cópia exata do bit-pattern (o valor contido na variável de origem) e entrega essa cópia para a variável de parâmetro do método. O método trabalha sempre com sua própria cópia local, sem nunca ter acesso direto à variável original da qual o argumento se originou.
</details>

4. **Objetos como Parâmetros:** Se Java é *pass-by-value*, como conseguimos alterar os atributos de um objeto passado para um método? O que exatamente está sendo passado "por valor"?
<details>
<summary>👀 Ver Resposta</summary>

O que está sendo passado por valor é o **endereço de referência** do objeto, e não o objeto em si. Quando você passa um objeto para um método, a JVM copia o endereço de memória que aponta para a Heap. Tanto a variável externa quanto o parâmetro local do método agora guardam cópias do mesmo endereço de referência e apontam para o mesmo objeto físico na Heap. Por isso, alterar os atributos através dessa referência afeta o objeto compartilhado. Porém, se dentro do método você reatribuir o parâmetro para um novo objeto (`param = new OutroObjeto()`), a variável original continuará apontando para o objeto anterior inalterada.
</details>

5. **Recursão:** O que caracteriza um método recursivo e qual é o principal risco se a recursão for mal projetada (relacionado à memória)?
<details>
<summary>👀 Ver Resposta</summary>

Um método recursivo é aquele que realiza uma ou mais chamadas a si mesmo para resolver subproblemas menores de um mesmo problema principal. O grande risco de uma recursão mal projetada (sem condição de término ou com chamadas excessivamente profundas) é o esgotamento da memória da pilha de chamadas (*Call Stack*), resultando no erro fatal `java.lang.StackOverflowError`.
</details>

6. **Caso Base (Base Case):** Qual a importância do caso base dentro de uma função recursiva?
<details>
<summary>👀 Ver Resposta</summary>

O caso base é a condição de parada da recursão: é o cenário mais simples do problema cuja resposta é conhecida e resolvida diretamente sem necessidade de invocar o método novamente. Ele é essencial porque impede que as chamadas a si mesmo se repitam indefinidamente, garantindo que a pilha de execução chegue a um fim e comece a desenrolar retornando os resultados calculados.
</details>

7. **Varargs (`...`):** Como funcionam os argumentos de comprimento variável em Java e qual é a regra sobre a sua posição na lista de parâmetros de um método?
<details>
<summary>👀 Ver Resposta</summary>

O recurso de Varargs (sintaxe `Tipo... nome`) permite que um método receba zero, um ou múltiplos argumentos daquele tipo separados por vírgula, ou um array já montado. Internamente, o compilador do Java converte o parâmetro varargs em um array convencional (`Tipo[]`). A regra sintática obrigatória é que um método só pode ter **no máximo um parâmetro varargs** e ele deve ser **obrigatoriamente o último parâmetro** declarado na lista de argumentos do método.
</details>

8. **Retorno de Métodos:** Qual é a diferença no fluxo de execução quando usamos o comando `return` em um método do tipo `void` vs um método com tipo de retorno especificado?
<details>
<summary>👀 Ver Resposta</summary>

* Em um método com tipo de retorno (ex: `int`, `String`), o comando `return valor;` é obrigatório em todos os caminhos possíveis de execução do código, encerrando imediatamente o método e entregando um dado útil de volta para quem o invocou.
* Em um método `void`, o `return;` (sem argumentos) é puramente opcional e atua como uma instrução de controle de fluxo de interrupção imediata (*early return*), finalizando o método antes de alcançar a última linha sem entregar nenhum valor de retorno.
</details>

9. **O Modificador `static`:** O que significa dizer que um método é estático? Em nível de memória e acesso, a quem ele pertence?
<details>
<summary>👀 Ver Resposta</summary>

Um método marcado como `static` pertence à **Classe como um todo**, e não a uma instância individual específica da classe. Ele pode ser invocado diretamente através do nome da classe (ex: `Math.sqrt(25)`) sem necessidade de criar nenhum objeto com `new`. Em termos de memória, ele reside na área de metadados da classe carregada pela JVM e não possui a referência implícita `this`.
</details>

10. **Uso do `static`:** Por que um método estático não pode acessar variáveis de instância (não estáticas) ou invocar métodos de instância da mesma classe diretamente, sem criar um objeto?
<details>
<summary>👀 Ver Resposta</summary>

Porque variáveis e métodos de instância só ganham existência física na memória Heap após a instanciação de um objeto concreto com o operador `new`. O método estático existe e pode ser chamado a qualquer momento, mesmo quando nenhum objeto daquela classe foi criado. Como o método estático não opera sobre nenhuma instância específica e não possui a palavra-chave `this`, o compilador não teria como saber os dados de qual objeto ele deveria acessar ou modificar.
</details>
