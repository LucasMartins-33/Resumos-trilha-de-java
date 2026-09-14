# Anotações de Estudo: Java Core - Capítulo 14 (Behavioral Diagrams)

> [!NOTE]
> Os Diagramas Comportamentais modelam o aspecto dinâmico do sistema, ou seja, como os objetos interagem e como o sistema se comporta ao longo do tempo ou em resposta a eventos.

---

## 1. Use Case Diagram (Diagrama de Casos de Uso)
O objetivo principal é capturar os **Requisitos Funcionais** mostrando como entidades externas interagem com o sistema para atingir objetivos. Tem alto nível de abstração.

* **Actor (Ator)**: Entidade externa (usuário, outro sistema, hardware). Representado por um boneco-palito.
* **Use Case (Caso de Uso)**: Funcionalidade específica (representado por uma elipse/oval).
* **System Boundary (Fronteira do Sistema)**: Caixa que envolve os casos de uso, demarcando o escopo do que é o sistema (os atores ficam de fora).

### Tipos de Relacionamentos
1. **Association**: Linha reta entre Ator e Caso de Uso (comunicação normal).
2. **Generalization**: Herança. Um ator "Gerente" herda de "Funcionário". Um caso de uso "Gerenciar Produtos Físicos" herda de "Gerenciar Produtos".
3. **Include (Inclusão)**: `<<include>>`. Ocorre quando um caso de uso A **sempre** precisa executar o caso de uso B. Faz parte do fluxo principal e esperado. Ex: `Fazer Pagamento` inclui `Atualizar Estoque`.
4. **Extend (Extensão)**: `<<extend>>`. Ocorre sob **condições específicas (opcional)**. O caso de uso base não precisa do estendido para funcionar. Ex: `Fazer Pagamento` estende `Proceder pro Checkout` apenas se o usuário escolher um financiamento especial.

---

## 2. Sequence Diagram (Diagrama de Sequência)
É o diagrama mais usado por desenvolvedores! Modela a ordem cronológica estrita da interação entre objetos/componentes de cima para baixo.

* **Lifeline (Linha de vida)**: Representa o objeto participando da interação. Desenhada como uma linha tracejada vertical.
* **Activation Bar**: Retângulo sobre a lifeline mostrando o período em que o objeto está ativo (processando algo).
* **Messages (Mensagens)**:
  * **Síncrona (Call)**: Seta de linha cheia. Bloqueia e aguarda resposta.
  * **Assíncrona**: Seta tracejada. Não bloqueia.
  * **Return**: Linha tracejada voltando. Traz a resposta.
  * **Self Message**: Seta fazendo um loop de volta para a mesma lifeline (método interno/recursão).
  * **Create / Destroy**: Instanciação (seta terminando na caixa do objeto) / Destruição (um grande `X` no fim da lifeline).

### Fragments (Lógica Condicional e Loops)
Usados para agrupar mensagens e aplicar lógica:
* **Alt**: Alternativa (`if/else`). O fluxo toma um de vários caminhos baseados em condições.
* **Opt**: Opcional (`if` simples). Ocorre se a condição for verdadeira.
* **Loop**: Repetição (enquanto a condição for verdadeira).
* **Par**: Paralelismo (execução assíncrona/threads).
* **Ref**: Referência a outro diagrama de sequência externo para evitar inchar o desenho.

---

## 3. Activity Diagram (Diagrama de Atividades)
Focado no *workflow* (fluxo de trabalho) e passos de execução. Assemelha-se a um fluxograma, porém com recursos para modelar execuções paralelas.
* **Diferença pro Seq. Diagram**: Sequência foca em *Objetos e Mensagens cronológicas*. Atividade foca nos *Passos do Processo em Alto Nível*.

* **Initial/Final Node**: Círculo preto (início) / Alvo (fim).
* **Decision vs Merge**: O *Decision* (losango) ramifica o fluxo. O *Merge* junta caminhos de volta num só, aguardando que *apenas um* deles chegue.
* **Fork vs Join**: O *Fork* (barra preta vertical/horizontal) divide um fluxo em múltiplos fluxos paralelos. O *Join* unifica fluxos paralelos, mas **bloqueia** aguardando que *todos* os fluxos paralelos terminem para continuar.
* **Swimlanes (Raias)**: Divide as atividades por responsabilidade, departamento ou ator (ex: Raia do Cliente, Raia do Estoque, Raia do Pagamento). Podem ser horizontais ou verticais.

---

## 4. State Machine Diagram (Máquina de Estados)
Modela as mudanças de estado de um objeto específico. Ajuda a substituir gigantescos blocos de `ifs/switches` no código.
* **Tipos**:
  * **Behavioral**: Foca na lógica interna do objeto (metódos que mudam os estados).
  * **Protocol**: Foca na comunicação com entidades externas (ex: máquina de estados de uma conexão de rede TCP).
* **State**: Situação de um objeto (ex: Rascunho, Publicado, No Carrinho). 
  * Pode ser *Simples*, *Composite* (contém sub-estados, como `Dirigindo` -> `Acelerando/Freando`) ou *History* (lembra o último sub-estado em que o objeto estava antes de pausar).
* **Guards vs Triggers (Fundamentais!)**:
  * **Guard**: Condição booleana que deve ser verdadeira. Fica entre colchetes. Ex: `[price > 100]`.
  * **Trigger**: O evento em si que causa a transição. Sem colchetes. Ex: `button_pressed`.

---

## 5. Communication Diagram (Antigo Collaboration Diagram)
Similar ao Diagrama de Sequência (você pode até converter um no outro em algumas ferramentas), porém foca na **topologia/arquitetura das conexões (Links)** e não na linha do tempo vertical.
* Os objetos ficam espalhados livremente (geralmente conectados num padrão cliente/servidor ou fornecedor/cliente).
* **Numeração Estrita**: Para saber a ordem das ações, você numera cada mensagem: `1`, `1.1`, `1.2`. 
* **Paralelismo**: Se mensagens ocorrem em paralelo (multithreading), usam-se letras na mesma ordem: `1.6A` e `1.6B`.

---

## 6. Timing Diagram (Diagrama de Tempo)
Parece um diagrama de sequência invertido. É extremamente focado em sistemas de Tempo Real, Telecomunicações e Sistemas Embarcados onde microssegundos importam.
* O eixo X é o **Tempo**. O eixo Y são as lifelines ou estados.
* Permite desenhar **Duration Constraints** (Restrições de Duração - ex: essa requisição HTTP deve durar de 50ms a 200ms) visualmente sobre a linha do gráfico.

---

## 7. Interaction Overview Diagram
Atua como um Diagrama de Atividades onde os "nós" (blocos) são na verdade referências (`ref`) a outros diagramas completos de interação (como Diagramas de Sequência ou Comunicação).
* **Objetivo**: Integrar cenários gigantescos em um macro-fluxo sem sobrecarregar quem está lendo.
* Usa a estrutura visual do diagrama de atividades (Decisões, Fork/Join, Initial/Final Nodes), mas em vez de passos simples, os blocos são diagramas inteiros.
