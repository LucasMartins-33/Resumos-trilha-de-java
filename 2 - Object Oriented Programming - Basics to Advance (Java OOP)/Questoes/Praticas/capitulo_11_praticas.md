📘 Capítulo 11: Padrões de Projeto GoF

**O Cenário:**
Sua equipe precisa unificar o desenvolvimento usando design patterns mundialmente reconhecidos.

Sua missão é codificar 10 cenários distintos aplicando padrões do GoF.

🟢 Atividade 11.1: Singleton
1. Crie a classe `ConexaoBanco`.
2. Esconda o construtor colocando-o como `private`.
3. Crie um método estático `getInstance()` que retorna a mesma instância única toda vez.

🟢 Atividade 11.2: Factory Method
1. Crie a interface `Notificacao` com o método `enviar()`.
2. Crie as implementações `NotificacaoEmail` e `NotificacaoSMS`.
3. Crie a classe `NotificacaoFactory` com o método estático `criar(String tipo)` que devolve o objeto correto baseado na string.

🟡 Atividade 11.3: Builder
1. Crie a classe `Pedido` com 10 atributos diferentes (alguns opcionais).
2. Crie a classe interna (ou externa) `PedidoBuilder`.
3. Implemente os métodos fluidos (ex: `comDesconto(double desc) { ... return this; }`) e finalize com `build()`.

🟡 Atividade 11.4: Prototype
1. Crie uma classe `InimigoRPG` pesada (ex: com array de 100 posições).
2. Faça a classe implementar a interface nativa `Cloneable`.
3. Sobrescreva o método `clone()` para instanciar rapidamente novos inimigos sem reprocessar tudo.

🟠 Atividade 11.5: Adapter
1. Você tem a interface `Motor` (seu sistema) e a classe `MotorEletricoAntigo` (biblioteca de terceiro incompatível).
2. Crie a classe `AdaptadorMotor` que implementa `Motor`, mas internamente possui (Composição) um `MotorEletricoAntigo` e delega as chamadas, traduzindo os métodos.

🟠 Atividade 11.6: Decorator
1. Crie a interface `Cafe` com `double getPreco()`. Implemente `CafeSimples` retornando `2.0`.
2. Crie a classe `CafeComLeite` que implementa `Cafe`, recebe um `Cafe` no construtor (Decorator) e retorna `cafe.getPreco() + 1.0`.

🔴 Atividade 11.7: Facade
1. Imagine os sistemas complexos: `Cpu`, `Memoria`, `Disco`.
2. Crie uma classe `ComputadorFacade` com um método simples `ligar()`.
3. Dentro desse método, faça a orquestração completa (ex: inicializa memória, lê setor de boot no disco, envia para CPU).

🔴 Atividade 11.8: Strategy
1. Crie a interface `EstrategiaImposto` com `calcular(double valor)`.
2. Crie `ImpostoBrasil` e `ImpostoEUA`.
3. Crie a classe `Carrinho` que recebe a Estratégia no construtor e chama calcular de forma polimórfica (sem `if/else`).

🔴 Atividade 11.9: State
1. Crie a interface `EstadoPedido`.
2. Implemente `EstadoNovo`, `EstadoEnviado`, `EstadoEntregue`.
3. Se você tentar chamar `cancelar()` em `EstadoEntregue`, o estado deve lançar erro. A regra de transição muda de estado (comportamento muda em tempo de execução).

🔴 Atividade 11.10: Observer
1. Crie a classe `CanalYoutube` (Subject) contendo uma lista de inscritos.
2. Crie a interface `Inscrito` (Observer) com o método `atualizar(String video)`.
3. Faça o Canal notificar todos os inscritos da lista com um `for` quando um vídeo novo for postado.
