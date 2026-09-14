# Questões Teóricas: Capítulo 2 - Fundamentos de Orientação a Objetos

1. **Definição de Objeto:** No contexto da programação, o que exatamente é um objeto e quais são as duas características principais que todo objeto possui?
<details>
<summary>👀 Ver Resposta</summary>

Um objeto é uma instância concreta de uma classe criada na memória (Heap). Ele modela uma entidade do mundo real ou conceitual e possui duas características fundamentais: **Estado** (seus atributos, campos ou propriedades, que representam os dados que o objeto armazena) e **Comportamento** (seus métodos ou funções, que definem as ações, operações e manipulações que o objeto é capaz de executar).
</details>

2. **Classe vs Objeto:** Explique a relação entre uma Classe e um Objeto fazendo um paralelo com uma planta arquitetônica e uma casa real.
<details>
<summary>👀 Ver Resposta</summary>

A **Classe** é o molde, gabarito ou planta arquitetônica: ela define a estrutura, atributos e comportamentos possíveis, mas não ocupa espaço habitável nem possui existência física real por si só. O **Objeto** é a casa construída a partir dessa planta: uma entidade física e tangível alocada na memória (Heap), com valores concretos em seus atributos (como cor da parede e número de cômodos). A partir de uma única classe (planta), é possível instanciar dezenas de objetos (casas) independentes.
</details>

3. **Estado e Comportamento:** Em uma classe Java, como nós representamos o "estado" (state) e o "comportamento" (behavior) de um objeto?
<details>
<summary>👀 Ver Resposta</summary>

O **Estado** é representado pelos **atributos (campos ou variáveis de instância)** da classe, responsáveis por armazenar os dados e características do objeto (ex: `private String titular; private double saldo;`). Já o **Comportamento** é representado pelos **métodos**, que contêm os blocos de código executáveis que alteram ou consultam o estado do objeto e interagem com outros elementos do sistema (ex: `public void depositar(double valor)`).
</details>

4. **Construtores:** O que é um método construtor? Quando ele é invocado e qual é o construtor padrão fornecido pelo Java caso nenhum seja definido?
<details>
<summary>👀 Ver Resposta</summary>

O construtor é um bloco especial de inicialização invocado no momento exato em que um novo objeto é instanciado na memória com o operador `new` (ex: `new ContaCorrente()`). Ele possui obrigatoriamente o mesmo nome da classe e não possui nenhum tipo de retorno (nem mesmo `void`). Se nenhum construtor for declarado na classe, o compilador do Java cria automaticamente um **construtor padrão** (*default constructor*), que não recebe argumentos e possui o corpo vazio. Caso o desenvolvedor declare qualquer construtor com parâmetros, o Java remove o construtor padrão automaticamente.
</details>

5. **A Palavra-chave `this`:** Para que serve a palavra-chave `this` dentro dos métodos e construtores de uma classe? Dê um exemplo de um problema que ela resolve.
<details>
<summary>👀 Ver Resposta</summary>

A palavra-chave `this` é uma referência que aponta para a **instância atual** do objeto que está executando o código. Ela resolve principalmente o problema de **sombreamento de variáveis** (*variable shadowing*), que acontece quando parâmetros de métodos ou construtores possuem nomes idênticos aos atributos da classe (ex: `this.saldo = saldo;`). Também serve para encadear construtores na mesma classe através de `this(...)` e para passar o próprio objeto como argumento para métodos externos.
</details>

6. **Instanciação e Heap:** Quando você usa a palavra-chave `new` em Java (ex: `new Carro()`), o que acontece fisicamente na memória (Heap) e o que a variável armazena?
<details>
<summary>👀 Ver Resposta</summary>

Ao executar `new Carro()`, a JVM aloca fisicamente um bloco de memória na **Heap** com tamanho suficiente para comportar todos os dados e atributos daquele objeto e executa o construtor correspondente. A variável que recebe esse retorno (ex: `Carro c`), que reside na memória **Stack** (se for variável local), armazena apenas um **endereço de referência** (um ponteiro gerenciado pela máquina virtual) que aponta para onde o objeto recém-criado reside na Heap.
</details>

7. **Tipos Primitivos vs Tipos de Referência:** Qual a diferença fundamental entre armazenar um `int` e armazenar uma referência para `Carro` em uma variável?
<details>
<summary>👀 Ver Resposta</summary>

Uma variável de tipo primitivo (como `int x = 10;`) armazena diretamente o **valor literal** no seu próprio espaço de memória na Stack. Já uma variável de tipo de referência (como `Carro c = new Carro();`) armazena apenas o **endereço de memória** apontando para o objeto na Heap. Copiar uma variável primitiva (`y = x`) clona o valor numérico; copiar uma referência (`c2 = c`) duplica o endereço de memória, fazendo com que ambas as variáveis passem a apontar para o mesmíssimo objeto na Heap.
</details>

8. **Variáveis Locais vs de Instância:** Qual a diferença de escopo e ciclo de vida entre uma variável declarada dentro de um método e uma declarada diretamente no corpo da classe?
<details>
<summary>👀 Ver Resposta</summary>

* **Variáveis Locais:** Declaradas dentro de métodos, blocos ou construtores. Seu escopo é restrito ao bloco de declaração e seu ciclo de vida dura apenas enquanto a execução daquele frame na Stack estiver ativa. O Java **não** as inicializa automaticamente; tentar lê-las sem atribuir valor prévio gera erro de compilação.
* **Variáveis de Instância (Campos):** Declaradas no corpo da classe fora dos métodos. Seu escopo abrange todos os métodos não-estáticos da classe e seu ciclo de vida está atrelado ao ciclo do próprio objeto na Heap (nascem no `new` e morrem na coleta do Garbage Collector). São inicializadas automaticamente pela JVM com valores padrão (`0`, `0.0`, `false`, `null`).
</details>

9. **Garbage Collection:** Em C e C++, os desenvolvedores precisam liberar memória manualmente. Em Java, isso não é necessário. Como o *Garbage Collector* do Java sabe que um objeto pode ser removido da memória?
<details>
<summary>👀 Ver Resposta</summary>

O Garbage Collector (GC) utiliza algoritmos baseados em **rastreabilidade** (*reachability analysis*) partindo de pontos iniciais conhecidos chamados **GC Roots** (como variáveis locais ativas na Stack, threads ativas e variáveis estáticas). Se um objeto na Heap não puder mais ser alcançado por nenhuma cadeia de referências partindo de uma GC Root (tornando-se inalcançável/órfão), ele é considerado lixo e sua memória é desalocada e reciclada automaticamente pelo GC.
</details>

10. **Pacotes (Packages):** Qual é o propósito dos pacotes em Java e como eles ajudam a evitar conflitos de nomes em projetos de grande escala?
<details>
<summary>👀 Ver Resposta</summary>

Os pacotes (`package`) organizam classes e interfaces em namespaces e pastas hierárquicas, estruturando o código por domínio funcional e permitindo o controle de visibilidade (nível *package-private*). Eles evitam colisões de nomes (*naming collisions*) porque o identificador completo de uma classe no Java é o seu **FQN** (*Fully Qualified Name*). Isso permite ter, por exemplo, `com.empresa.pagamento.Pedido` e `com.empresa.entrega.Pedido` coexistindo no mesmo sistema sem ambiguidades.
</details>
