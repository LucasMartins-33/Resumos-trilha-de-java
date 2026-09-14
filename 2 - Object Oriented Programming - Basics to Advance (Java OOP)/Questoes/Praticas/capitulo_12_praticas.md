📘 Capítulo 12: Desafios Avançados de Design Patterns

**O Cenário:**
Um arquiteto sênior pediu para você implementar padrões arquiteturais avançados e provar que você não vai cair nas "armadilhas" comuns (Anti-patterns).

🟢 Atividade 12.1: Consertando o Singleton (Thread-safe)
1. Revise seu `Singleton` da aula anterior.
2. E se duas Threads acessarem `getInstance()` ao exato mesmo tempo quando a instância é `null`? Ambas vão criar objetos (Race condition).
3. Conserte usando bloco `synchronized` (Double-checked locking).

🟢 Atividade 12.2: Template Method (Hollywood Principle)
1. Crie uma classe abstrata `ProcessadorPagamento` com o método principal fixo `final void processar() { validar(); debitar(); notificar(); }`.
2. Deixe `validar` e `debitar` abstratos para as subclasses implementarem, mas `notificar` concreto. A superclasse dita o fluxo.

🟡 Atividade 12.3: O Anti-pattern God Object
1. Identifique uma classe no seu sistema fictício que tenha mais de 1000 linhas, métodos de banco, de tela e de arquivo.
2. Escreva um comentário propondo como dividi-la usando o padrão Facade + Strategy.

🟡 Atividade 12.4: Command (Undo / Redo)
1. Crie a interface `Comando` com métodos `executar()` e `desfazer()`.
2. Crie `ComandoAdicionarTexto`.
3. Crie um Histórico (Stack) e guarde os comandos ali para habilitar um "CTRL+Z".

🟠 Atividade 12.5: Flyweight (Otimização Extrema)
1. Em um jogo com 1 milhão de árvores, você não pode instanciar um modelo 3D para cada uma.
2. Crie a classe `ArvoreModelo` (Flyweight) contendo o 3D, e uma `ArvorePlantada` que só contém as coordenadas X, Y apontando para o mesmo modelo.

🟠 Atividade 12.6: Bridge (Separando Dimensões)
1. Você tem Controles Remotos (Básico, Avançado) e Dispositivos (TV, Radio). Se você usar herança pura, terá que fazer TVBasica, TVAvançada, RadioBasico...
2. Use Bridge: Controle recebe no construtor a Interface `Dispositivo`. Desacople!

🔴 Atividade 12.7: Chain of Responsibility
1. Crie classes de suporte técnico: `Nivel1`, `Nivel2`, `Nivel3`.
2. Cada classe aponta para o próximo nível. Se o Nivel1 não sabe resolver o `Bug`, repassa para o Nivel2.
3. Implemente esse fluxo recursivo elegante de tratamento de problemas.

🔴 Atividade 12.8: Composite (Estrutura de Árvore)
1. Você tem Pastas e Arquivos. Pastas podem conter outras Pastas ou Arquivos.
2. Crie a interface `ComponenteDeSistema`.
3. Ambos, `Pasta` e `Arquivo` implementam, mas a `Pasta` guarda uma lista de `ComponenteDeSistema`. Calcule o tamanho total chamando `tamanho()` recursivamente na raiz.

🔴 Atividade 12.9: Memento (Salvar Estado Seguramente)
1. Crie um objeto `EditorDeTexto`.
2. Em vez do Histórico ler variáveis privadas do Editor (violando encapsulamento), crie a classe interna `Memento` que o Editor gera para salvar seu próprio estado e restaurá-lo.

🔴 Atividade 12.10: Null Object Pattern
1. É irritante colocar `if (cliente != null)` em todo lugar.
2. Crie uma classe `ClienteNulo` que herda de `Cliente` ou implementa sua interface, mas cujos métodos (como `pagar()`) simplesmente não fazem nada.
3. Retorne essa instância em vez de `null` nos repositórios.
