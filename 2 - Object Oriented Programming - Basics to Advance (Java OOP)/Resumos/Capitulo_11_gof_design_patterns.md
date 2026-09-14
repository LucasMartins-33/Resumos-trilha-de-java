# Anotações de Estudo: Java Core - Capítulo 11 (GoF Design Patterns)

> [!NOTE]
> Os padrões de projeto **GoF (Gang of Four)** vêm do livro de 1994 escrito por Erich Gamma, Richard Helm, Ralph Johnson e John Vlissides. São 23 padrões que fornecem soluções reutilizáveis e testadas pelo tempo para problemas comuns em OOP. 
> Eles são divididos em três grandes categorias:
> 1. **Criacionais (Creational)**: Lidam com o mecanismo de criação de objetos.
> 2. **Estruturais (Structural)**: Como classes e objetos são compostos para formar estruturas maiores.
> 3. **Comportamentais (Behavioral)**: Interação e divisão de responsabilidades entre os objetos.

---

## 1. Padrões Criacionais (Creational Patterns)

### 1.1. Singleton
Garante que apenas uma instância de uma classe exista e provê um ponto de acesso global a ela.
* **Uso**: Quando não precisamos criar instâncias repetidas (ex: `OrderManagementService` sem estado).
* **Estrutura**: Campo privado estático, construtor privado, método acessor estático público (com inicialização preguiçosa - *lazy* - e sincronizado para ambientes multithread).
* **Dica de Ouro**: O Singleton só se torna um anti-pattern se guardar **estado compartilhado** mutável, o que dificulta testes e gera bugs. Singleton "Stateless" (sem estado) é seguro. Em frameworks modernos (Spring), a inversão de controle gerencia o ciclo de vida como Singleton por padrão.

### 1.2. Prototype
Cria novos objetos copiando/clonando um protótipo, em vez de usar `new`.
* **Uso**: Quando instanciar um objeto é muito complexo/custoso computacionalmente.
* **Detalhe Crítico**: Requer **Deep Cloning** (Clonagem Profunda). O `.clone()` nativo do Java faz *Shallow Copy*. Use bibliotecas como `SerializationUtils` (Apache Commons) para clonagem profunda real via serialização.

### 1.3. Factory Method
Define uma interface para criar objetos, mas deixa a decisão de qual classe instanciar para as subclasses (ou lógica interna).
* **Modernização Java**: Em vez de ter blocos gigantes de `if/else` ou `switch` para instanciar tipos, o instrutor usou de forma brilhante referências de métodos e `Supplier`:
```java
// Exemplo atualizado (Factory moderno no Java)
private static final Map<String, Supplier<Archiver>> map = new HashMap<>();
static {
    map.put("zip", ZipArchiver::new);
    map.put("rar", RarArchiver::new);
}
public Archiver getArchiver(String type) {
    Supplier<Archiver> supplier = map.get(type);
    return supplier != null ? supplier.get() : null;
}
```

### 1.4. Builder
Constrói objetos complexos passo a passo.
* **Variação 1 (Canonical com Diretor)**: Um `Director` orquestra diferentes instâncias de `Builder` abstraídos (ex: `CheapComputerBuilder`, `ExpensiveComputerBuilder`).
* **Variação 2 (Chain Builder - O mais popular)**: 
```java
// Estrutura clássica moderna
Account acc = Account.newBuilder()
                     .withName("Lucas")
                     .withBalance(100)
                     .build();
```
* **Como fazer**: Classe interna estática `Builder` com métodos "setter" que retornam `this` (a própria instância do Builder) e um método terminador `build()` que devolve o objeto principal.

### 1.5. Abstract Factory
Cria "famílias" de objetos relacionados (ex: `MacOsWindow`, `MacOsButton` vs `WinWindow`, `WinButton`) sem especificar suas classes concretas.
* **Diferença pro Factory Method**: O Factory Method cria *um* produto. A Abstract Factory cria uma *família de múltiplos produtos* interconectados.

---

## 2. Padrões Estruturais (Structural Patterns)

*(Nota: O instrutor dividiu em duas partes, porém abordou apenas três explicitamente na Parte 2: Bridge, Flyweight e Composite. Proxy, Decorator, Adapter e Facade foram citados como parte 1).*

### 2.1. Bridge
Desacopla uma Abstração de sua Implementação para que ambas possam variar independentemente.
* **Exemplo**: `RemoteControl` (Abstração) pode ser `BasicRemote` ou `AdvancedRemote`. O aparelho, `Device` (Implementação), pode ser `TV` ou `Radio`. O `Remote` recebe o `Device` via construtor. Podemos alterar/adicionar novos controles remotos sem tocar na lógica das TVs.

### 2.2. Flyweight
Reduz o consumo de memória compartilhando dados idênticos (estado intrínseco) entre múltiplos objetos, deixando o estado único (estado extrínseco) de fora.
* **Exemplo**: Renderizar 1 milhão de árvores na tela. 
  * **Estado Intrínseco (Compartilhado)**: Cor, textura, modelo 3D (Fica na classe `TreeType` que atua como Flyweight).
  * **Estado Extrínseco (Único)**: Coordenadas X e Y. (Passado como argumento ao desenhar).
* **Fábrica**: O `FlyweightFactory` guarda as instâncias num mapa em cache para reuso.

### 2.3. Composite
Agrupa objetos em estruturas de árvore e trata objetos individuais e grupos de maneira uniforme.
* **Exemplo**: Uma forma geométrica (`Shape`). Um grupo de formas (`CompoundShape`) herda da mesma interface `Shape` e possui uma lista de `Shape`s internamente. Chamar `.draw()` no `CompoundShape` repassa a chamada em recursão para todos os filhos, sem o cliente precisar usar loops visíveis.

---

## 3. Padrões Comportamentais (Behavioral Patterns)

### 3.1. Strategy
Encapsula uma família de algoritmos, tornando-os intercambiáveis.
* **Uso**: Compressão de arquivos (Zip vs Rar), métodos de pagamento, ordenação.
* **Java Moderno**: No Java, interfaces do tipo Strategy geralmente são *Functional Interfaces* (`@FunctionalInterface`), permitindo que a estratégia seja injetada usando Lambdas em vez de instanciar novas classes. (ex: `list.sort((a, b) -> a - b)`).

### 3.2. Command
Encapsula uma requisição como um objeto (Comando) contendo um método `.execute()`.
* **Uso**: Enfileiramento de requisições, botões de UI, ou operações de "Desfazer" (Undo).
* **Estrutura**: Invoker (Quem chama) -> Command (O encapsulador) -> Receiver (Quem faz a lógica, ex: `Light.turnOn()`).

### 3.3. Template Method
Define o esqueleto (ordem de execução) de um algoritmo em uma classe abstrata, permitindo que subclasses sobrescrevam passos específicos (*Hooks*).
* **Uso**: Processos padronizados com pequenas variações (Ex: Compilador cross-platform onde "Alocar RAM" é igual, mas "Gerar executável" difere entre iOS e Android).

### 3.4. Iterator
Permite acessar sequencialmente elementos de um objeto agregado sem expor sua representação estrutural subjacente (se é array, lista ligada, hash).
* **Métodos chaves**: `hasNext()`, `next()`. Presente massivamente na API de *Collections* do Java.

### 3.5. Chain of Responsibility
Passa a requisição por uma "corrente" de objetos processadores.
* **Exemplo**: Caixa eletrônico (Notas de 50 -> Notas de 20 -> Notas de 10) ou *Filtros Web* (Filtro de Login -> Filtro de Autorização -> Controller). Se um elo não sabe lidar, repassa ao `.next()`.

### 3.6. Visitor
Separa um algoritmo da estrutura do objeto no qual ele opera. Permite adicionar novos comportamentos a classes existentes sem alterá-las.
* **Mecânica Crítica (*Double Dispatch*)**: Elementos possuem o método `accept(Visitor v) { v.visit(this); }`.
* **Diferença pro Decorator**: Decorator altera *um objeto* específico. Visitor aplica lógicas a uma *árvore/grupo de classes variadas*. Ex: Exportador XML/JSON (O Visitor) lendo dados de diferentes locais (`Bank`, `School`, `House`).

### 3.7. State
Permite que um objeto mude de comportamento quando seu estado interno muda, parecendo ter alterado sua classe original.
* **Solução**: Em vez de `ifs` ou `switches`, criamos classes diferentes para cada estado (`DraftState`, `PublishedState`) que implementam uma interface comum. O objeto de Domínio apenas delega as ações para a classe de estado atual dele.

### 3.8. Observer (Pub/Sub)
Define relação 1:N onde um objeto (Subject/Publisher) muda de estado e todos os seus dependentes (Observers/Subscribers) são notificados.
* **Uso**: Avisar clientes de que um produto chegou em estoque sem forçá-los a verificar manualmente (polling).

### 3.9. Memento
Captura e restaura o estado interno de um objeto (Undo/Redo), sem violar o encapsulamento.
* **Tríade**: 
  1. `Originator`: O objeto principal que gera o *snapshot*.
  2. `Memento`: O objeto com os dados básicos salvos (imutável).
  3. `Caretaker`: Controla o histórico (lista de Mementos).

### 3.10. Interpreter
Avalia uma gramática ou linguagem específica (DSL). Trabalha criando uma *Abstract Syntax Tree (AST)* contendo Expressões Finais (*Terminal*) e Intermediárias (*Non-terminal*), que alteram um Contexto global.

### 3.11. Mediator
Reduz o acoplamento caótico entre vários componentes centralizando a comunicação em um objeto Mediador.
* **Uso**: Componentes complexos de UI. Um checkbox não precisa conhecer um dropdown list. Ambos conhecem o "Mediator" (Dialog/Controller), que ouve o checkbox e manda o dropdown se esconder.
