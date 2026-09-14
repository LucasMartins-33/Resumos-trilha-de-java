# Questões Teóricas: Capítulo 7 - Preparação para Entrevistas OOP

1. **Classes Abstratas vs Interfaces (Java 8+):** Com a introdução dos *default methods*, as interfaces em Java ganharam poder. Qual a diferença fundamental de uso e propósito arquitetural que ainda existe entre Classes Abstratas e Interfaces?
<details>
<summary>👀 Ver Resposta</summary>

Mesmo com métodos `default`, as diferenças fundamentais permanecem em **Estado** e **Identidade**:
* **Estado:** Classes abstratas podem ter atributos de instância mutáveis (`private int idade`), construtores para inicializar campos e blocos de inicialização. Interfaces só podem conter constantes públicas estáticas (`public static final`).
* **Identidade e Herança:** Classes abstratas modelam identidade estrita ("É um" / *is-a*), e uma classe Java só pode estender uma única classe. Interfaces modelam capacidades ou papéis ("Comporta-se como" / *can-do*), e uma classe pode implementar dezenas de interfaces simultaneamente.
</details>

2. **Coesão:** Como você definiria Coesão de forma simples numa entrevista técnica?
<details>
<summary>👀 Ver Resposta</summary>

Coesão é a medida do grau de foco e alinhamento das responsabilidades de uma classe ou módulo. Uma classe com **alta coesão** possui um propósito único e bem delimitado, onde todos os seus atributos e métodos trabalham em conjunto para cumprir uma mesma meta de negócio. Classes de baixa coesão tentam fazer tarefas diversas e não correlatas, tornando o código difícil de manter e testar.
</details>

3. **Acoplamento:** Por que o alto acoplamento é considerado um "code smell" grave em sistemas grandes?
<details>
<summary>👀 Ver Resposta</summary>

O alto acoplamento (*tight coupling*) ocorre quando classes dependem profundamente dos detalhes internos e implementações concretas de outras classes. Isso é um code smell grave porque cria o efeito de "falha em dominó": qualquer alteração simples em uma classe propaga quebras em cascata por todo o sistema. Além disso, impede o isolamento para testes unitários automatizados e torna praticamente impossível reutilizar componentes em outros contextos.
</details>

4. **Agregação vs Composição:** Na prática do código, como diferenciamos uma relação de Agregação de uma de Composição? (Considere o ciclo de vida dos objetos envolvidos).
<details>
<summary>👀 Ver Resposta</summary>

A diferenciação reside na independência do **ciclo de vida**:
* **Composição (Vínculo Forte):** O objeto-parte não tem sentido de existir sem o objeto-todo; seus ciclos de vida coincidem. Se o objeto-todo for destruído, o objeto-parte morre junto. No código, a parte geralmente é instanciada diretamente dentro do todo (ex: `Carro` instancia seu próprio `Motor`).
* **Agregação (Vínculo Fraco):** O objeto-parte possui existência e ciclo de vida totalmente independentes do todo. No código, a parte geralmente é criada fora e passada via construtor ou setter para o todo (ex: `Departamento` recebe uma lista de `Professores`; se o departamento for fechado, os professores continuam existindo no sistema).
</details>

5. **A palavra-chave `final`:** Explique os 3 efeitos distintos da palavra `final` quando aplicada a: Variáveis, Métodos e Classes.
<details>
<summary>👀 Ver Resposta</summary>

* **Variável `final`:** Transforma a variável em constante ou imutável em referência. Seu valor só pode ser atribuído uma única vez e nunca mais alterado (se for um objeto, a referência não pode apontar para outro, embora o estado interno do objeto ainda possa mudar).
* **Método `final`:** Proíbe que qualquer subclasse sobrescreva (`@Override`) o método, garantindo que o algoritmo original permaneça imutável.
* **Classe `final`:** Proíbe totalmente a herança da classe (nenhuma outra classe pode usar `extends` a partir dela). Exemplos clássicos são as classes `String` e `Integer`.
</details>

6. **A Classe `Object`:** Toda classe em Java herda implicitamente da classe `Object`. Cite e explique o propósito de 3 métodos essenciais herdados dessa classe.
<details>
<summary>👀 Ver Resposta</summary>

1. **`equals(Object obj)`:** Avalia a igualdade lógica entre dois objetos (por padrão compara apenas referências de memória `==`, mas costuma ser sobrescrito para comparar valores de atributos).
2. **`hashCode()`:** Retorna um inteiro de hash representando o objeto para uso em estruturas de alta performance baseadas em tabelas hash (como `HashMap` e `HashSet`). Deve seguir rigorosamente o contrato com o método `equals`.
3. **`toString()`:** Retorna uma representação textual legível do objeto, muito usada em logs, depuração e exibição (por padrão retorna `NomeDaClasse@hashcodeHexadecimal`).
</details>

7. **Herança Múltipla:** Qual é o argumento principal (técnico) pelo qual os criadores do Java optaram por não suportar herança múltipla de classes?
<details>
<summary>👀 Ver Resposta</summary>

O argumento técnico principal foi a **eliminação da complexidade e da ambiguidade**, notadamente o **Problema do Diamante** (*Diamond Problem*). Em linguagens como C++, a herança múltipla gera incertezas sobre qual estado ou método herdar quando duas classes pais compartilham o mesmo ancestral, além de introduzir problemas complexos de ponteiros de tabelas virtuais (vptrs/vtables) e dependências cíclicas difíceis de depurar.
</details>

8. **Early Binding vs Late Binding:** Qual é a diferença entre ligação estática (resolvida em tempo de compilação) e ligação dinâmica (resolvida em tempo de execução)? Qual conceito de OOP utiliza o *Late Binding*?
<details>
<summary>👀 Ver Resposta</summary>

* **Early Binding (Ligação Estática):** Ocorre em tempo de compilação, quando o compilador já sabe exatamente qual código de método será executado (usado em métodos `private`, `static`, `final` e na sobrecarga de métodos).
* **Late Binding (Ligação Dinâmica):** Ocorre em tempo de execução, onde a JVM analisa o objeto real alocado na Heap para decidir qual implementação executar. O conceito central de OOP que utiliza Late Binding é o **Polimorfismo de Sobrescrita** (*Dynamic Method Dispatch*).
</details>

9. **Impedindo a Herança:** Além de usar o modificador `final`, existe alguma outra técnica baseada em construtores para impedir que uma classe seja instanciada ou herdada por código de terceiros?
<details>
<summary>👀 Ver Resposta</summary>

Sim: declarar todos os **construtores da classe como `private`**. Como qualquer subclasse precisa obrigatoriamente chamar o construtor da superclasse via `super()` na primeira linha de seu próprio construtor, uma subclasse não conseguirá compilar se todos os construtores da classe pai forem privados. Essa técnica é a base do padrão **Singleton** e de classes utilitárias que contêm apenas métodos estáticos (como `java.lang.Math`).
</details>

10. **Method Hiding:** O que acontece quando uma subclasse declara um método estático com a mesma assinatura de um método estático da superclasse? Isso é polimorfismo?
<details>
<summary>👀 Ver Resposta</summary>

Isso é chamado de **Ocultação de Método** (*Method Hiding*) e **NÃO é polimorfismo**. Métodos estáticos pertencem à classe e são resolvidos em tempo de compilação (*Early Binding*) com base no tipo da variável de referência, e não no objeto em execução. Se você declarar `Pai p = new Filho();` e chamar `p.metodoEstatico()`, a JVM executará a versão do método da classe `Pai`, provando que não há despacho dinâmico.
</details>
