# Questões Teóricas: Capítulo 11 - Padrões de Projeto GoF

1. **Categorias dos Padrões GoF:** Quais são as 3 categorias fundamentais em que os 23 padrões de projeto da Gang of Four (GoF) são divididos e qual o propósito de cada uma?
<details>
<summary>👀 Ver Resposta</summary>

1. **Padrões Criacionais (Creational):** Focam nos mecanismos e processos de criação e instanciação de objetos, tornando o sistema independente de como seus objetos são criados, compostos e representados (ex: Singleton, Factory Method, Builder).
2. **Padrões Estruturais (Structural):** Tratam de como classes e objetos são montados e compostos para formar estruturas maiores e mais flexíveis, mantendo-as eficientes (ex: Adapter, Decorator, Facade).
3. **Padrões Comportamentais (Behavioral):** Tratam da comunicação, atribuição de responsabilidades, algoritmos e coordenação do fluxo de trabalho entre objetos (ex: Strategy, Observer, State).
</details>

2. **Singleton:** Além de garantir apenas uma instância, qual outro recurso (frequentemente considerado negativo) o Singleton provê no sistema globalmente?
<details>
<summary>👀 Ver Resposta</summary>

O Singleton provê um **ponto de acesso global** àquela instância em qualquer parte da aplicação (atuando como uma variável global disfarçada). Isso é frequentemente considerado negativo porque introduz um forte acoplamento oculto, dificulta testes unitários (já que o estado global persiste entre os testes e impede mocks fáceis) e pode causar problemas de concorrência em ambientes multithread.
</details>

3. **Factory Method vs Abstract Factory:** Qual é a diferença fundamental em escopo e intenção entre esses dois padrões de criação?
<details>
<summary>👀 Ver Resposta</summary>

* **Factory Method:** Baseia-se em **herança de classes**. Ele define um método abstrato em uma classe base delegando para subclasses a decisão sobre qual classe concreta instanciar (focado na criação de um único produto).
* **Abstract Factory:** Baseia-se em **composição de objetos**. Ele fornece uma interface para criar **famílias inteiras de objetos relacionados ou dependentes** (ex: botões, janelas e caixas de texto com o mesmo tema visual) sem que o código precise especificar suas classes concretas.
</details>

4. **Builder:** Como o padrão Builder resolve o temido "Anti-pattern do Telescoping Constructor" (quando uma classe tem múltiplos construtores com variações de argumentos)?
<details>
<summary>👀 Ver Resposta</summary>

O padrão Builder separa o processo de construção de um objeto complexo de sua representação final. Em vez de criar múltiplos construtores com diferentes combinações de parâmetros (o que gera código confuso e propenso a passar parâmetros no lugar errado), o Builder oferece uma API fluente de métodos encadeados (ex: `.setNome("X").setIdade(20).build()`), tornando a construção legível, personalizável e permitindo que objetos finais imutáveis sejam construídos de forma segura.
</details>

5. **Prototype:** Em quais cenários faz mais sentido clonar objetos existentes (via padrão Prototype) do que instanciá-los usando o operador `new`?
<details>
<summary>👀 Ver Resposta</summary>

Faz sentido quando a criação de um novo objeto do zero via `new` for **computacionalmente muito custosa**, envolver operações pesadas de I/O (leitura de arquivos, consultas ao banco de dados ou requisições de rede) ou exigir configurações de estado complexas que já foram resolvidas em um objeto protótipo existente na memória.
</details>

6. **Adapter:** O padrão Adapter (ou Wrapper) permite que objetos incompatíveis colaborem. Descreva uma situação do mundo real em sistemas onde um Adapter seria vital.
<details>
<summary>👀 Ver Resposta</summary>

Um cenário clássico ocorre na **integração de gateways de pagamento**: seu sistema possui uma interface padrão `ProcessadorDePagamento` com o método `pagar(BigDecimal valor)`. No entanto, você precisa contratar um serviço de terceiros (como PayPal ou Stripe) cuja biblioteca externa possui métodos completamente diferentes, como `executeTransaction(PaymentPayload payload)`. O Adapter implementa a sua interface interna e traduz as chamadas para os métodos da biblioteca externa, permitindo a integração sem modificar o código do seu sistema.
</details>

7. **Decorator:** O padrão Decorator anexa responsabilidades adicionais a um objeto dinamicamente. Por que isso é uma alternativa superior a gerar múltiplas subclasses para expandir funcionalidades?
<details>
<summary>👀 Ver Resposta</summary>

Porque a herança estática gera uma **explosão combinatória de subclasses** (ex: `CafeComLeite`, `CafeComCanela`, `CafeComLeiteECanela`). O Decorator envolve o objeto base em classes decoradoras dinamicamente em tempo de execução através de composição, permitindo combinar comportamentos de forma modular e ilimitada sem inflar a hierarquia de classes. Um exemplo consagrado é a biblioteca de I/O do Java (`new BufferedReader(new InputStreamReader(System.in))`).
</details>

8. **Facade:** Qual a principal intenção do padrão Facade ao encobrir a complexidade de múltiplos subsistemas ou bibliotecas sob uma única interface simplificada?
<details>
<summary>👀 Ver Resposta</summary>

A intenção é prover uma **interface de alto nível simples e unificada** para um ecossistema complexo de classes ou subsistemas. Em vez de o cliente precisar interagir diretamente com dezenas de classes, inicializações e dependências complexas (ex: codecs de áudio, descompactadores de vídeo, mixers de som), ele interage com uma única classe Facade que expõe um método direto (ex: `reproduzirVideo(arquivo)`), reduzindo drasticamente o acoplamento.
</details>

9. **Strategy vs State:** Ambos encapsulam algoritmos ou comportamentos e permitem trocá-los em tempo de execução. Mas em termos de QUEM decide essa troca de comportamento, qual a principal diferença conceitual entre Strategy e State?
<details>
<summary>👀 Ver Resposta</summary>

* **Strategy:** A troca de comportamento é geralmente **decidida de fora**, pelo cliente ou contexto que escolhe conscientemente qual estratégia injetar (ex: escolher entre frete Sedex ou Jadlog). As estratégias são independentes e geralmente não conhecem umas às outras.
* **State:** As mudanças de comportamento ocorrem **internamente** como consequência de transições de estado do próprio objeto. Os estados concretos costumam ter conhecimento de outros estados e controlam a transição entre eles conforme eventos ocorrem (ex: um `Pedido` transita automaticamente de `AguardandoPagamento` para `Pago` e depois para `Enviado`).
</details>

10. **Observer:** Como o padrão Observer (Publisher/Subscriber) promove um forte baixo acoplamento temporal e estrutural entre o sujeito que emite um evento e os objetos que o escutam?
<details>
<summary>👀 Ver Resposta</summary>

O sujeito (*Subject/Publisher*) mantém apenas uma lista de referências genéricas para a interface `Observer`. Quando seu estado muda, ele simplesmente itera pela lista chamando o método `update()` em cada observador. O sujeito não precisa saber quem são os observadores concretos, o que eles farão com a notificação, nem quantos são, permitindo que novos observadores se inscrevam ou cancelem sua inscrição a qualquer momento sem nenhuma alteração no sujeito.
</details>
