# Questões Teóricas: Capítulo 12 - Entrevista sobre Design Patterns

1. **Composição sobre Herança:** Os padrões GoF são famosos pela máxima "Favoreça a composição de objetos sobre a herança de classes". Quais são as maiores desvantagens da herança de classes que justificam essa afirmação?
<details>
<summary>👀 Ver Resposta</summary>

As maiores desvantagens da herança são:
1. **Acoplamento forte e quebra de encapsulamento:** Subclasses ficam expostas a detalhes da implementação da superclasse (*Fragile Base Class*);
2. **Rigidez em tempo de compilação:** A relação é estática e não pode ser trocada em tempo de execução;
3. **Explosão combinatória de classes:** Tentar cobrir combinações de comportamentos via herança gera dezenas de classes desnecessárias;
4. **Herança inadequada de comportamento:** A subclasse herda todos os métodos públicos do pai, inclusive os que não fazem sentido para ela, violando o LSP.
</details>

2. **Anti-patterns:** O que são *Anti-patterns* e qual é o impacto do chamado "God Object" ou "Blob" em um sistema?
<details>
<summary>👀 Ver Resposta</summary>

*Anti-patterns* são soluções recorrentes para problemas comuns de software que parecem atraentes inicialmente, mas que geram consequências altamente negativas para o design e manutenção. O **God Object** (ou Blob) é uma classe centralizada que acumula a maioria dos dados e responsabilidades do sistema, transformando todas as outras classes em meros sacos de dados anêmicos. Seu impacto inclui extrema dificuldade de leitura, testes unitários inviáveis e risco contínuo de regressões a cada alteração.
</details>

3. **Singleton e Concorrência:** O Singleton clássico muitas vezes falha em ambientes multi-thread. Qual é a principal armadilha técnica (race condition) ao implementar um Singleton de forma ingênua?
<details>
<summary>👀 Ver Resposta</summary>

A armadilha ocorre quando duas threads executam simultaneamente a verificação de inicialização preguiçosa `if (instance == null)`. Se a thread A avaliar a condição como verdadeira e sofrer uma pausa de CPU antes de instanciar o objeto, a thread B entrará na mesma condição e criará uma instância. Quando a thread A retomar, ela também criará uma segunda instância, quebrando a unicidade do Singleton. A solução técnica recomendada em Java é utilizar o **Double-Checked Locking com a variável marcada como `volatile`** ou recorrer a classes estáticas internas (*Bill Pugh Singleton*) ou `enum`.
</details>

4. **Hollywood Principle (Inversão de Controle):** O padrão **Template Method** implementa a máxima "Não nos chame, nós chamaremos você". Como ele consegue forçar a ordem do algoritmo e ainda assim permitir customização pelas subclasses?
<details>
<summary>👀 Ver Resposta</summary>

O Template Method define o esqueleto geral do algoritmo dentro de um método concreto (geralmente marcado como `final` na superclasse). Esse método orquestra a chamada de etapas em uma sequência estrita. Algumas etapas são métodos abstratos ou ganchos (*hooks*) que as subclasses são obrigadas ou convidadas a sobrescrever. Dessa forma, as subclasses preenchem os detalhes do comportamento, mas é a superclasse que controla quando e como essas etapas serão executadas.
</details>

5. **Bridge vs Adapter:** A intenção do *Adapter* é fazer duas coisas prontas funcionarem juntas após o design. Já o padrão *Bridge* é projetado *antes*. O que o padrão Bridge tenta separar?
<details>
<summary>👀 Ver Resposta</summary>

O padrão Bridge desacopla intencionalmente **uma Abstração de sua Implementação**, de modo que ambas possam variar de forma totalmente independente. Ele é projetado preventivamente antes da codificação para evitar que duas dimensões ortogonais de variação (por exemplo: tipos de janelas de UI e tipos de sistemas operacionais onde elas rodam) multipliquem subclasses hierárquicas, conectando-as por uma ponte de composição.
</details>

6. **Padrão Command:** Como o padrão Command transforma uma "chamada de método" em um objeto tangível, e como isso facilita a implementação de históricos e funcionalidades de "Undo/Redo"?
<details>
<summary>👀 Ver Resposta</summary>

O padrão Command encapsula uma solicitação, com todos os seus parâmetros, contexto e receptor, dentro de um objeto autônomo que implementa uma interface comum (com métodos como `execute()` e `undo()`). Como as ações agora são objetos de primeira classe na memória, elas podem ser armazenadas em listas, empilhadas em estruturas de dados de histórico (*Undo Stack*) e serializadas, permitindo reverter ou reexecutar comandos sequencialmente com facilidade.
</details>

7. **Flyweight vs Object Pool:** Ambos economizam memória/recursos. A diferença é que um *Pool* lida com instâncias que não podem ser compartilhadas simultaneamente, enquanto o *Flyweight* lida com compartilhamento estrutural. Explique como o Flyweight separa estados intrínsecos e extrínsecos.
<details>
<summary>👀 Ver Resposta</summary>

* **Estado Intrínseco:** É a parte dos dados do objeto que é invariável, compartilhável e independente de contexto (ex: a textura, malha 3D e propriedades de uma árvore em um jogo). Esse estado fica guardado dentro do objeto Flyweight compartilhado por milhares de referências.
* **Estado Extrínseco:** É a parte variável que depende do contexto exclusivo daquele elemento específico (ex: as coordenadas X, Y, Z onde aquela árvore está posicionada no mapa). Esse estado é mantido fora do Flyweight e passado como argumento para seus métodos quando necessário.
</details>

8. **Padrões e OCP:** Como o padrão **Strategy** adere perfeitamente ao Princípio do Aberto/Fechado (OCP) do SOLID na prática?
<details>
<summary>👀 Ver Resposta</summary>

O Strategy extrai algoritmos variantes para classes separadas que implementam uma mesma interface comum. A classe cliente depende apenas dessa interface abstrata. Quando uma nova regra de negócio ou algoritmo surgir, basta criar uma nova classe concreta implementando a interface, sem modificar nenhuma linha de código da classe cliente ou das estratégias existentes, atendendo de forma exemplar ao OCP.
</details>

9. **Mediator:** Como o padrão Mediator alivia a comunicação caótica ("teia de aranha" - *many-to-many*) entre objetos, tornando os relacionamentos uma estrela (*star topology*)?
<details>
<summary>👀 Ver Resposta</summary>

Em vez de cada componente manter referências diretas e comunicar-se com múltiplos outros componentes (criando uma teia de acoplamento mútuo *many-to-many*), todos os componentes se comunicam exclusivamente com um único objeto central: o **Mediator**. O Mediator recebe os eventos de cada componente e decide para quais outros elementos a mensagem deve ser repassada, transformando o fluxo em uma topologia em estrela com baixo acoplamento.
</details>

10. **Sindrome do Arquiteto:** Em que situações a aplicação de um Design Pattern complexo (como uma arquitetura de múltiplos Factories e Decorators) é na verdade uma má escolha?
<details>
<summary>👀 Ver Resposta</summary>

Quando aplicada preventivamente a problemas simples que não exigem essa flexibilidade (*Overengineering* ou complexidade acidental). Se os requisitos forem estáveis, o volume de variações for nulo e o escopo do projeto for reduzido, introduzir padrões sofisticados apenas adiciona camadas desnecessárias de indireção, dificulta a leitura do código para a equipe e aumenta o custo de manutenção sem gerar benefícios concretos.
</details>
