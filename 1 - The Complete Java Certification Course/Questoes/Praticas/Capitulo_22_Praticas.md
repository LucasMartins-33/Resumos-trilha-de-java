# Questões Práticas - Capítulo 22 (Concurrency and Multithreading)

🟢 Nível 1: A Criação Clássica (`extends Thread`)
Cenário: Você quer um processo rodando em segundo plano imprimindo números.
Sua Tarefa: 
* Crie a estrutura de uma classe `ContadorNumeros` que herde de `Thread`.
* Sobrescreva obrigatoriamente o método `public void run()`.
* Dentro dele, faça um loop de 1 a 10 imprimindo os números.

🟡 Nível 2: Dando o Gatilho
Cenário: Você precisa dar "vida" a Thread do nível 1 e comprovar o paralelismo.
Sua Tarefa:
* Dentro de uma classe executável (com método `main`), instancie o seu `ContadorNumeros`.
* Dê o comando de disparo do paralelismo `.start()`.
* Imediatamente abaixo, adicione outro loop (rodando na Main Thread) que imprima "Main rodando" 10 vezes. Se o computador for justo, os prints das duas threads se misturarão!

🟠 Nível 3: A Estrutura Limpa (`implements Runnable`)
Cenário: Você descobriu que `ContadorNumeros` precisa herdar regras da classe `Relogio` futuramente, então você não pode "queimar" o extends com `Thread`.
Sua Tarefa:
* Mude a assinatura da sua classe `ContadorNumeros` para implementar a interface `Runnable`.
* No método `main`, note que `.start()` sumiu do `ContadorNumeros`.
* Para contornar, crie um "veículo" instanciando um `new Thread(...)` passando o seu contador como parâmetro, e então dispare o start pela thread.

🔴 Nível 4: A Pausa Dramática (`Thread.sleep`)
Cenário: O contador está muito rápido.
Sua Tarefa:
* Dentro do loop do método `run()`, coloque a instrução `Thread.sleep(1000);` (Pausa de 1 segundo a cada iteração).
* Lembre-se, o `sleep` assusta o Java. Você será obrigado a envolver a linha inteira num bloco de `try / catch (InterruptedException)` para caso o sistema feche abruptamente e interrompa o "sono" da sua thread.

🟣 Nível 5: O Conflito de Acesso (Race Condition)
Cenário: O departamento de TI criou a variável `int bancoDeDados = 0`. Duas threads farão `bancoDeDados++` simultaneamente num loop infinito.
Sua Tarefa (Apenas Teórica / Observacional):
* O que acontece com os dados após 10 segundos rodando se esse incremento não for protegido? (R: A contagem ficará menor do que o real, pois atualizações colidirão e anularão a matemática umas das outras).

🟤 Nível 6: O Cadeado Universal (Synchronized Method)
Cenário: Vamos blindar a operação crítica.
Sua Tarefa:
* Crie uma classe `GestorEstoque` com um método `public void somarProduto()`.
* Adicione o modificador/palavra-chave necessária antes do `void` para que o Java nunca deixe mais de 1 thread pisar dentro desse método ao mesmo tempo. 

🔵 Nível 7: O Bloco Cirúrgico (Synchronized Block)
Cenário: O método `somarProduto()` cresceu. Agora ele faz cálculos de impostos pesados ANTES de somar no banco. Se trancarmos o método inteiro, as threads ficarão congestionadas esperando matemática inútil.
Sua Tarefa:
* Retire o `synchronized` da assinatura do método.
* Isole as operações (ex: `this.estoque++`) envolvendo-as em um bloco cirúrgico `synchronized(this) { ... }`.
* Deixe a matemática de impostos fora do bloco. A performance explodirá.

🟢 Nível 8: A Magia da BlockingQueue
Cenário: Você está montando o motor de Processamento de Pagamentos (Produtor x Consumidor) mais estável e enxuto possível.
Sua Tarefa:
* Declare uma variável `BlockingQueue<String> fila = new ArrayBlockingQueue<String>(50);` (Com limite travado em 50 pagamentos).
* Dentro do código Produtor (Thread A), você jogará itens usando o método `fila.put("Boleto_1")`. Se a fila tiver 50, essa thread dormirá até esvaziar.
* Dentro do código Consumidor (Thread B), você extrairá itens usando `fila.take()`. Se a fila zerar, essa thread dormirá aguardando boletos. *Lógica 100% livre de synchronized manual!*

🟡 Nível 9: O Chefe das Tarefas (ExecutorService)
Cenário: A Black Friday começou e a loja tem 1.000 clientes (Runnable Tasks) para serem processados. Não vamos abrir 1.000 threads.
Sua Tarefa:
* No `main`, use o pacote Concurrent para declarar um Pool limitador de Threads.
* Crie: `ExecutorService executor = Executors.newFixedThreadPool(10);` (Cria uma piscina restrita a 10 trabalhadores apenas).
* Simule a injeção do trabalho chamando `executor.execute(suaTarefaRunnable)` ou, caso seja um array de tarefas, injetando as 1000 num loop. As 10 threads darão conta das 1000 sozinhas.

🟠 Nível 10: O Botão de Desligar (Shutdown)
Cenário: As 1000 tarefas da Black Friday do Nível 9 terminaram na piscina, mas a tela preta do console não fecha. A aplicação ficou viva no Limbo!
Sua Tarefa:
* Após empurrar (execute) os códigos para a fila do executor, adicione a linha de código imperativa para destruir o `ExecutorService` de forma macia e garantir que o Java morra em paz no final das execuções: `executor.shutdown();`.
