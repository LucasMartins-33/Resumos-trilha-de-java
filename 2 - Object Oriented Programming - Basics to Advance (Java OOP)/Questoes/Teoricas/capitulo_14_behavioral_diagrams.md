# Questões Teóricas: Capítulo 14 - Diagramas Comportamentais (Behavioral)

1. **Casos de Uso (Use Cases):** Qual é o foco primário de um Diagrama de Casos de Uso? Ele se preocupa mais em documentar *o que* o sistema faz ou *como* ele faz internamente?
<details>
<summary>👀 Ver Resposta</summary>

O foco primário do Diagrama de Casos de Uso é documentar as interações entre os atores externos e o sistema a partir da perspectiva do usuário. Ele se preocupa estritamente com **O QUE** o sistema faz (as funcionalidades e metas que oferece aos atores), e **NÃO com COMO** ele faz internamente (ignorando algoritmos, estruturas de dados, classes e tabelas de banco de dados).
</details>

2. **Include vs Extend:** No Casos de Uso, a relação `<<include>>` significa que a execução de um caso de uso é obrigatória para o caso primário. Qual é a condição para que um caso de uso conectado por `<<extend>>` seja executado?
<details>
<summary>👀 Ver Resposta</summary>

A execução de um caso de uso conectado por `<<extend>>` é **opcional e condicional**. Ele só é acionado se uma condição pré-determinada for satisfeita em um ponto de extensão específico (*Extension Point*) do fluxo primário (ex: o caso de uso "Calcular Frete Expresso" só é executado no caso de uso "Comprar Produto" se o cliente optar voluntariamente por entrega prioritária).
</details>

3. **Diagrama de Atividades (Activity Diagram):** Muitas vezes comparado a um fluxograma, qual é o principal diferencial visual do Diagrama de Atividades que o torna ideal para modelar fluxos concorrentes/paralelos?
<details>
<summary>👀 Ver Resposta</summary>

Seu principal diferencial é o suporte nativo a **bifurcações e sincronizações paralelas** através das barras de **Fork** e **Join**, além da organização visual em **Raias de Piscina (*Swimlanes*)**, que distribuem visualmente as atividades entre diferentes participantes, departamentos ou microsserviços responsáveis pela execução.
</details>

4. **Forks e Joins:** No diagrama de atividades, para que servem as barras grossas (Forks e Joins) no meio do fluxo de execução?
<details>
<summary>👀 Ver Resposta</summary>

* **Fork:** Divide um único fluxo de controle em dois ou mais fluxos paralelos que passam a executar de forma concorrente e assíncrona.
* **Join:** Atua como ponto de sincronização onde múltiplos fluxos paralelos convergem; o fluxo posterior ao Join só pode prosseguir depois que **todos** os fluxos de entrada tiverem concluído suas tarefas.
</details>

5. **Diagrama de Sequência (Sequence Diagram):** Este é o diagrama mais utilizado da UML para interações. O que o eixo horizontal (X) e o eixo vertical (Y) representam em um Diagrama de Sequência?
<details>
<summary>👀 Ver Resposta</summary>

* **Eixo Horizontal (X):** Representa os participantes ou instâncias dos objetos/serviços envolvidos na interação (sem ordem temporal no eixo X).
* **Eixo Vertical (Y):** Representa a **passagem do tempo**, lida de cima para baixo, documentando a cronologia exata em que as mensagens são disparadas e recebidas.
</details>

6. **Lifelines e Execution Specifications:** O que são as linhas de vida tracejadas (Lifelines) e as caixas retangulares (Execution Occurrences) que aparecem sobre elas?
<details>
<summary>👀 Ver Resposta</summary>

* **Linha de Vida (Lifeline):** É a linha vertical tracejada que desce a partir da caixa do objeto, representando a existência daquele participante durante o intervalo de tempo da interação.
* **Barra de Ativação (Execution Occurrence / Activation Bar):** É o retângulo fino desenhado sobre a linha de vida, indicando o período exato em que aquele objeto está executando uma operação ou processando uma chamada.
</details>

7. **Mensagens Síncronas vs Assíncronas:** No Diagrama de Sequência, como diferenciamos visualmente uma mensagem síncrona (ex: uma chamada de método normal) de uma mensagem assíncrona (ex: jogar uma mensagem numa fila)?
<details>
<summary>👀 Ver Resposta</summary>

* **Mensagem Síncrona:** Desenhada com uma **linha sólida com ponta de seta preenchida (triangular sólida)** ($\longrightarrow$), indicando que o remetente pausa e aguarda a resposta do destinatário.
* **Mensagem Assíncrona:** Desenhada com uma **linha sólida com ponta de seta aberta (duas linhas formando um ângulo)** ($\longrightarrow$), indicando que o remetente dispara a mensagem (como um evento no Kafka) e continua seu processamento imediatamente sem esperar.
</details>

8. **Combined Fragments (Fragmentos Combinados):** Como representar lógica de controle, como um bloco `if-else` ou um laço de repetição (`for`), dentro de um Diagrama de Sequência?
<details>
<summary>👀 Ver Resposta</summary>

Utilizam-se caixas retangulares sobrepostas às linhas de vida chamadas **Fragmentos Combinados**, com uma etiqueta (*operator*) no canto superior esquerdo:
* **`alt`:** Para lógica condicional com alternativas (`if / else`);
* **`opt`:** Para um bloco condicional opcional (`if` simples sem else);
* **`loop`:** Para repetições e laços (`for`, `while`), contendo uma condição de guarda entre colchetes.
</details>

9. **Diagrama de Máquina de Estados:** O que compõe uma Máquina de Estados? Qual é a diferença essencial entre "Estado" e "Transição"?
<details>
<summary>👀 Ver Resposta</summary>

Uma Máquina de Estados modela os ciclos de vida de uma entidade por meio de Estados, Eventos e Transições:
* **Estado:** É uma condição ou situação estável na vida de um objeto na qual ele atende a determinados critérios ou aguarda por um evento (ex: `PENDENTE`, `PAGO`).
* **Transição:** É a mudança dinâmica e atômica de um estado de origem para um novo estado de destino, provocada pela ocorrência de um gatilho/evento e sujeita a condições de guarda (ex: o evento `confirmarPagamento()` transita o pedido de `PENDENTE` para `PAGO`).
</details>

10. **Escolha de Diagrama:** Se você precisa desenhar o ciclo de vida completo pelo qual um "Pedido de Venda" passa (Novo, Pago, Enviado, Concluído) incluindo os eventos que disparam essas mudanças, qual diagrama comportamental é o ideal?
<details>
<summary>👀 Ver Resposta</summary>

O diagrama ideal é o **Diagrama de Máquina de Estados** (*State Machine Diagram*), pois ele é focado especificamente em expressar as fases discretas que um único objeto transita ao longo do tempo em resposta a eventos de negócio, além de explicitar transições inválidas ou proibidas.
</details>
