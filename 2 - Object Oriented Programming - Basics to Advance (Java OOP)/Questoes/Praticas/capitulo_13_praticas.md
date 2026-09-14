📘 Capítulo 13: Introdução à UML na Prática (PlantUML)

**O Cenário:**
Muitas empresas modernas escrevem diagramas usando código (Text-to-UML) com ferramentas como PlantUML ou Mermaid, o que permite versionar no Git.

Sua missão é praticar os comandos básicos descritivos de UML. (Escreva em comentários como ficaria a notação visual ou use notação PlantUML/Mermaid fictícia).

🟢 Atividade 13.1: Esboço de Classe Base
1. Escreva a notação UML para representar a classe `Carro` com atributos privados (`-`) `marca` e público (`+`) método `ligar()`.

🟢 Atividade 13.2: Representando Herança UML
1. Escreva a notação text-to-uml para mostrar que a classe `Caminhao` herda de `Veiculo` (no PlantUML usa-se a seta `<|--`).

🟡 Atividade 13.3: Representando Realização (Interface)
1. Escreva como se representa a interface `Diriqivel` e a implementação dela pela classe `Carro` (geralmente linha tracejada `<|..`).

🟡 Atividade 13.4: Diagrama como Documentação (Sketch)
1. Descreva o conceito: Qual é a principal diferença de rigor entre esboçar um UML num quadro branco numa reunião ágil vs tentar fazer um UML que gere o código de produção?

🟠 Atividade 13.5: Diagrama Estrutural (Pacotes)
1. Desenhe (ou descreva) 3 pacotes principais: `Controller`, `Service`, `Repository`.
2. Crie setas pontilhadas de dependência indo do Controller para Service, e de Service para Repository.

🟠 Atividade 13.6: Diagrama Comportamental Inicial
1. Descreva a notação de Ator (um "boneco de palito") em UML.
2. Associe-o a um Caso de Uso chamado (Comprar Produto), geralmente representado por um formato oval.

🔴 Atividade 13.7: Estereótipos `<< >>`
1. Em uma modelagem, precisamos indicar que certa classe não é comum, mas sim uma "Entidade do Banco de Dados".
2. Adicione o estereótipo `<<Entity>>` acima do nome da classe no seu desenho mental ou código UML.

🔴 Atividade 13.8: Diferenciando Diagramas
1. Se o CTO pedir "Mostre-me os nós físicos dos servidores e qual componente roda onde", qual o diagrama correto que você selecionará no catálogo da UML? (Resp: Implantação).
2. Justifique a escolha.

🔴 Atividade 13.9: UML vs Código
1. Modifique a classe `Carro` no diagrama para incluir uma propriedade fortemente tipada: `motor: Motor`.
2. Explique como a UML é agnóstica à linguagem, representando listas como `motor: Motor[1..*]` ao invés de `List<Motor>`.

🔴 Atividade 13.10: Composição de Diagramas
1. Junte o ator, o componente físico (nó de implantação) e uma classe do banco em uma breve dissertação explicando como os vários diagramas da UML contam juntos a história total do seu software, cada um fornecendo uma "visão" (view) diferente.
