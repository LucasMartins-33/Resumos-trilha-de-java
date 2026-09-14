# Questões Teóricas: Capítulo 3 - Pilares da Orientação a Objetos

1. **Encapsulamento:** O que significa encapsular os dados de uma classe? Quais modificadores de acesso e métodos padrão do Java são geralmente usados para isso?
<details>
<summary>👀 Ver Resposta</summary>

Encapsulamento é a prática de ocultar o estado interno e os detalhes de implementação de um objeto, restringindo o acesso direto de código externo aos seus campos. Em Java, isso é implementado declarando os atributos com o modificador `private` e expondo operações controladas por meio de métodos públicos (`public`), comumente chamados de **Getters e Setters** (ou métodos de negócio expressivos). Isso garante que regras de validação, integridade e consistência sejam sempre aplicadas antes de qualquer alteração de estado.
</details>

2. **Herança:** A herança promove a reutilização de código, mas ao custo de introduzir qual problema estrutural entre a classe pai e a classe filha?
<details>
<summary>👀 Ver Resposta</summary>

A herança introduz um **forte acoplamento estrutural** (*tight coupling*) entre a superclasse e a subclasse. Esse problema decorre do fato de que qualquer alteração na implementação da classe pai pode quebrar inadvertidamente comportamentos assumidos pelas classes filhas (fenômeno conhecido como *Fragile Base Class Problem*), além de expor detalhes internos da superclasse violando o encapsulamento.
</details>

3. **Polimorfismo:** Explique a diferença entre polimorfismo em tempo de compilação (estático) e em tempo de execução (dinâmico).
<details>
<summary>👀 Ver Resposta</summary>

* **Polimorfismo Estático (Tempo de Compilação):** Ocorre através da **Sobrecarga de Métodos** (*Method Overloading*). O compilador determina qual versão do método invocar analisando o nome e a lista de argumentos (quantidade, tipos e ordem) antes da execução do programa.
* **Polimorfismo Dinâmico (Tempo de Execução):** Ocorre através da **Sobrescrita de Métodos** (*Method Overriding*). Ocorre quando uma subclasse reimplementa um método herdado de uma superclasse ou interface. A decisão de qual implementação executar é feita pela JVM em tempo de execução com base no tipo concreto do objeto instanciado na Heap (*Late Binding* / *Dynamic Dispatch*), e não no tipo da variável de referência.
</details>

4. **Abstração:** Qual é a principal diferença conceitual e prática entre uma `Classe Abstrata` e uma `Interface`?
<details>
<summary>👀 Ver Resposta</summary>

* **Classe Abstrata:** Representa uma relação estrita de identidade ("É um" / *is-a*). Serve como base para uma hierarquia de herança (`extends`), podendo conter estado (atributos de instância mutáveis), construtores, blocos de código e uma mistura de métodos abstratos e concretos. Uma classe Java só pode herdar de uma única classe abstrata.
* **Interface:** Representa um contrato de capacidade ou comportamento ("Faz papel de" / "Comporta-se como" / *can-do*). Foca em definir o que deve ser feito sem guardar estado mutável (seus campos são implicitamente `public static final`). Uma classe Java pode implementar (`implements`) múltiplas interfaces, proporcionando maior flexibilidade e baixo acoplamento.
</details>

5. **Modificadores de Acesso:** Explique a visibilidade conferida por `private`, `default` (package-private), `protected` e `public`.
<details>
<summary>👀 Ver Resposta</summary>

* **`private`:** Visível **apenas dentro da própria classe** onde foi declarado.
* **`default` (Package-Private - sem modificador explícito):** Visível por qualquer classe que esteja **dentro do mesmo pacote** (diretório).
* **`protected`:** Visível por classes do **mesmo pacote** e também por **subclasses** em qualquer pacote por meio de herança.
* **`public`:** Visível por **todas as classes** de qualquer pacote do projeto.
</details>

6. **Sobrescrita e `@Override`:** Por que é uma boa prática sempre utilizar a anotação `@Override` ao sobrescrever um método da superclasse?
<details>
<summary>👀 Ver Resposta</summary>

A anotação `@Override` instrui o compilador a verificar formalmente se o método anotado está de fato sobrescrevendo um método existente na superclasse ou interface. Caso ocorra um erro de digitação no nome do método ou uma incompatibilidade na lista de parâmetros, o compilador emitirá um **erro de compilação imediato**. Sem essa anotação, o compilador interpretaria o método como uma nova sobrecarga (*overload*), introduzindo bugs silenciosos e difíceis de rastrear.
</details>

7. **O Problema do Diamante:** O que é o *Diamond Problem* (Problema do Diamante) relacionado à herança múltipla?
<details>
<summary>👀 Ver Resposta</summary>

O Problema do Diamante surge em linguagens que permitem herança múltipla de classes (como C++) quando uma classe `D` herda simultaneamente de `B` e `C`, e ambas herdam da mesma superclasse `A`. Se `B` e `C` sobrescreverem um mesmo método de `A`, a classe `D` enfrenta uma ambiguidade insoluvel: o compilador não sabe qual implementação de método (`B` ou `C`) deve ser herdada por `D`.
</details>

8. **Herança Múltipla em Java:** Como o Java consegue mitigar os problemas da herança múltipla permitindo a implementação de múltiplas interfaces?
<details>
<summary>👀 Ver Resposta</summary>

O Java proíbe a herança múltipla de classes (uma classe só pode herdar de uma única superclasse), eliminando o risco de herdar múltiplos estados ou implementações conflitantes. No entanto, o Java permite implementar **múltiplas interfaces**, pois tradicionalmente interfaces contêm apenas contratos abstratos cuja única implementação reside na própria classe concreta. Desde o Java 8 (com métodos `default`), se duas interfaces tiverem métodos default idênticos, o compilador obriga a classe que as implementa a sobrescrever explicitamente o método conflitante, eliminando qualquer ambiguidade.
</details>

9. **Casting:** Qual a diferença entre *Upcasting* e *Downcasting* em hierarquias de classes? Qual deles pode gerar um erro de runtime (`ClassCastException`)?
<details>
<summary>👀 Ver Resposta</summary>

* **Upcasting:** É a conversão de uma referência de um tipo filho para um tipo pai (ex: `Animal a = new Cachorro();`). É sempre **seguro e implícito**, pois todo cachorro é garantidamente um animal.
* **Downcasting:** É a conversão de uma referência de um tipo pai para um tipo mais específico (filho) na hierarquia (ex: `Cachorro c = (Cachorro) a;`). Exige sintaxe explícita com parênteses e **pode lançar `ClassCastException` em tempo de execução** se o objeto real na Heap não for do tipo de destino (ou de uma subclasse dele). Para prevenir esse erro, recomenda-se verificar o tipo previamente com o operador `instanceof`.
</details>

10. **Composição vs Herança:** Por que há um princípio famoso em Design de Software que diz "Favoreça Composição sobre Herança" (*Favor composition over inheritance*)?
<details>
<summary>👀 Ver Resposta</summary>

A herança cria um acoplamento estático e rígido decidido em tempo de compilação (relação "É um"), onde a classe filha depende da implementação interna da classe pai. Já a **composição** estabelece uma relação flexível de "Tem um", onde uma classe delega tarefas para instâncias de outras classes passadas via construtor ou setter. A composição permite trocar comportamentos dinamicamente em tempo de execução, facilita a criação de testes unitários com mocks e evita o problema das classes base frágeis.
</details>
