# Questões Teóricas: Capítulo 15 - Diagramas Estruturais (Structural)

1. **Diagrama de Classes:** Quais são os 3 compartimentos (seções) essenciais na notação retangular de uma Classe na UML?
<details>
<summary>👀 Ver Resposta</summary>

Os 3 compartimentos verticais de uma classe na UML são:
1. **Compartimento Superior:** Nome da Classe (além de estereótipos como `<<interface>>` ou marcação em itálico se for abstrata);
2. **Compartimento Central:** Atributos da classe (com visibilidade, nome e tipo de dado);
3. **Compartimento Inferior:** Métodos e operações da classe (com visibilidade, nome, parâmetros e tipo de retorno).
</details>

2. **Visibilidade (Visibility):** Na notação do Diagrama de Classes, os símbolos `+`, `-`, `#` e `~` antes dos atributos representam quais modificadores de acesso, respectivamente?
<details>
<summary>👀 Ver Resposta</summary>

* **`+` :** `public` (público - visível de qualquer lugar);
* **`-` :** `private` (privado - visível apenas dentro da classe);
* **`#` :** `protected` (protegido - visível no mesmo pacote e em subclasses);
* **`~` :** `default` ou *package-private* (visível apenas por classes do mesmo pacote).
</details>

3. **Associação (Association):** Em um Diagrama de Classes, o que indica visualmente que a relação entre duas classes é uma associação fraca versus uma dependência efêmera (onde a classe não salva a instância como atributo global)?
<details>
<summary>👀 Ver Resposta</summary>

* **Associação:** É desenhada com uma **linha contínua sólida**, indicando que a classe mantém uma referência estrutural duradoura para o outro objeto (geralmente como um atributo de instância da classe).
* **Dependência:** É desenhada com uma **linha tracejada com ponta de seta aberta**, indicando uma relação efêmera (o objeto apenas usa temporariamente a outra classe como parâmetro de método, tipo de retorno ou variável local sem guardá-la em atributo).
</details>

4. **Multiplicidade (Multiplicity):** O que significam marcações como `0..1` ou `1..*` nas pontas das linhas de associação e como isso orienta a codificação de listas ou objetos opcionais em Java?
<details>
<summary>👀 Ver Resposta</summary>

Multiplicidades definem a cardinalidade mínima e máxima da relação:
* **`0..1`:** Indica que a relação é opcional (zero ou no máximo uma instância). Em Java, mapeia-se tipicamente como um campo simples que pode ser nulo ou envelopado em um `Optional<T>`.
* **`1..*`:** Indica que deve existir no mínimo uma e potencialmente muitas instâncias associadas. Em Java, mapeia-se como uma coleção (ex: `List<T>`, `Set<T>`) acompanhada de uma regra de validação que impede que a coleção esteja vazia.
</details>

5. **Agregação vs Composição (Visual):** Visualmente, qual a diferença das setas/linhas entre um relacionamento de Agregação e um de Composição?
<details>
<summary>👀 Ver Resposta</summary>

* **Agregação (Todo/Parte Fraco):** Desenhada com um **losango vazio (não preenchido)** na ponta da linha conectada à classe "todo" (ex: $\diamondsuit$—).
* **Composição (Todo/Parte Forte):** Desenhada com um **losango preenchido (sólido)** na ponta da linha conectada à classe "todo" (ex: $lacklozenge$—).
</details>

6. **Diagrama de Objetos:** Se o Diagrama de Classes atua como a planta arquitetônica, o Diagrama de Objetos atua como uma "fotografia". O que exatamente um Diagrama de Objetos documenta que um de Classes não pode?
<details>
<summary>👀 Ver Resposta</summary>

O Diagrama de Objetos documenta o **estado real e concreto da memória em um determinado instante de execução** (*snapshot*). Ele mostra instâncias nomeadas específicas, os valores exatos preenchidos em cada um dos seus atributos (ex: `saldo = 150.00`) e as conexões de referência ativas entre esses objetos específicos, o que é impossível em um Diagrama de Classes, que lida apenas com regras abstratas.
</details>

7. **Diagrama de Componentes:** Qual o propósito prático do Diagrama de Componentes e como as portas e conectores no estilo *Ball and Socket* (interfaces providas e requeridas) definem a comunicação?
<details>
<summary>👀 Ver Resposta</summary>

O propósito é modelar a arquitetura física e lógica de subsistemas modulares de software (JARs, serviços, bibliotecas) e suas dependências. 
A notação **Ball and Socket** (*bola e soquete*) define contratos de interface de forma elegante:
* **Ball (Círculo completo):** Representa uma **Interface Provida** (o componente oferece aquele serviço para o sistema).
* **Socket (Semicírculo):** Representa uma **Interface Requerida** (o componente precisa consumir aquele serviço para funcionar). Conectar uma bola a um soquete demonstra acoplamento limpo através de interfaces.
</details>

8. **Diagrama de Pacotes (Packages):** Como o Diagrama de Pacotes ajuda arquitetos a visualizar e evitar dependências circulares em sistemas gigantes de escala corporativa?
<details>
<summary>👀 Ver Resposta</summary>

Ele agrupa subsistemas em pastas conceituais e explicita as setas de dependência entre eles. Isso permite identificar visualmente e eliminar **Dependências Circulares** (onde o pacote A depende do pacote B, que por sua vez depende do pacote A direta ou indiretamente), garantindo uma arquitetura limpa e hierárquica baseada no Princípio das Dependências Aclíclicas (*Acyclic Dependencies Principle - ADP*).
</details>

9. **Diagrama de Implantação (Deployment):** Qual a diferença clara documentada por esse diagrama entre um *Node* (Nó) e um *Artifact* (Artefato)? Dê um exemplo prático de cada um.
<details>
<summary>👀 Ver Resposta</summary>

* **Node (Nó):** Representa um recurso físico ou virtual de hardware ou infraestrutura de execução (desenhado como um cubo 3D). Exemplo prático: `Servidor AWS EC2`, `Cluster Kubernetes` ou `Instância de Banco de Dados PostgreSQL`.
* **Artifact (Artefato):** Representa o arquivo físico de software compilado que é implantado dentro do nó (desenhado como um retângulo com estereótipo `<<artifact>>`). Exemplo prático: `app-financeiro.jar` ou `imagem-docker-v2.tar`.
</details>

10. **Composite Structure Diagram:** Para classes altamente complexas ou subsistemas que encapsulam muita lógica (ex: a Placa-mãe de um PC, contendo CPU, Memória internamente interligados), por que usar a Estrutura Composta é melhor do que um simples Diagrama de Classes?
<details>
<summary>👀 Ver Resposta</summary>

Porque o Diagrama de Classes tradicional mostra relacionamentos de forma genérica entre classes, mas não consegue expressar **como as partes internas de um componente se conectam entre si em um contexto restrito**. O Diagrama de Estrutura Composta permite "abrir a tampa" do componente complexo e detalhar suas portas de comunicação, partes internas colaboradoras e conectores internos sem poluir a visibilidade global do restante do sistema.
</details>
