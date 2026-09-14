# Questões Teóricas - Capítulo 22 (Concurrency and Multithreading)

**1. O Conceito Base:** O que é o conceito de Multithreading no Java e qual é a principal vantagem de utilizá-lo em comparação com a execução sequencial (Main Thread única) de um programa?
<details>
<summary>👀 Ver Resposta</summary>

Multithreading é a capacidade de um programa criar ramificações (fios de execução) para realizar múltiplas tarefas de forma simultânea/paralela. A vantagem é o aumento absurdo de performance e responsividade, pois o programa não precisa travar e esperar uma tarefa demorada (como baixar um arquivo) terminar para começar a executar outras coisas.
</details>

**2. A Criação de Threads:** Quais são as duas maneiras arquiteturais de se criar uma Thread no Java e qual delas é geralmente preferida pelos desenvolvedores (e por quê)?
<details>
<summary>👀 Ver Resposta</summary>

1) Herdando a classe `Thread` (`extends Thread`). 2) Implementando a interface `Runnable`. A segunda opção é amplamente preferida porque o Java proíbe herança múltipla. Se a sua classe herdar de `Thread`, ela nunca mais poderá herdar de nenhuma outra classe de negócios (como `extends Funcionario`). A interface deixa o caminho livre.
</details>

**3. O Erro de Iniciação:** Se você instanciar uma Thread (ex: `Thread t1 = new Thread(tarefa);`) e chamar a linha `t1.run();` diretamente, o que o Java fará? Qual é o método correto para dar vida ao paralelismo?
<details>
<summary>👀 Ver Resposta</summary>

Se você chamar `.run()`, o Java executará o método de forma totalmente sequencial na Main Thread atual, agindo como um método comum e travando o seu programa. Para que a mágica aconteça, você **deve chamar o `.start()`**. Ele avisará a JVM para criar um fio paralelo novo e, de lá de dentro, o Java chamará o `run()` simultaneamente.
</details>

**4. Intersecção Fatal (Interleaving):** O que é o problema de *Interleaving* em multithreading quando não usamos segurança/sincronização no acesso a variáveis?
<details>
<summary>👀 Ver Resposta</summary>

Ocorre quando duas threads independentes leem a mesma variável ao mesmo tempo e ambas tentam modificá-la antes que a outra termine (Condição de Corrida / Race Condition). O resultado é que os dados se atropelam e o valor final gravado será incorreto (geralmente uma atualização é esmagada pela outra).
</details>

**5. A Chave Mestra (Synchronized):** Como a palavra-chave `synchronized` age para proteger um bloco de código crítico ou um método contra os problemas do Multithreading?
<details>
<summary>👀 Ver Resposta</summary>

Ela atua como um "cadeado" (Lock). Quando a primeira Thread entra num bloco `synchronized`, ela tranca a porta. Se outras threads chegarem, elas serão congeladas (ficarão aguardando do lado de fora). Apenas quando a Thread 1 terminar tudo e sair do bloco, a porta se destranca para a próxima entrar. Isso garante a execução atômica (ininterrupta) daquele trecho.
</details>

**6. Bloco vs Método Sincronizado:** Em termos de performance, por que arquitetos de software frequentemente preferem usar Blocos Sincronizados (`synchronized(this) { ... }`) em vez de colocar a palavra `synchronized` direto na assinatura de um método gigante?
<details>
<summary>👀 Ver Resposta</summary>

Porque o método inteiro tranca o objeto desde a linha 1 até a última, criando um gargalo enorme e atrasando todas as threads (matando o propósito do paralelismo). Com o bloco, você tranca **apenas** as linhas de código restritas que realmente manipulam a variável crítica, deixando o restante da lógica pesada livre para todas as threads acessarem juntas.
</details>

**7. Coleções Inseguras:** Se o seu sistema rodar várias threads que injetam dados num `ArrayList` ou `HashMap` comum ao mesmo tempo, qual erro o Java provavelmente lançará no console? Onde encontramos as coleções blindadas contra isso?
<details>
<summary>👀 Ver Resposta</summary>

O Java muito possivelmente vomitará a exceção `ConcurrentModificationException`. Para coleções em paralelo, nunca use as versões normais; importe as versões Thread-Safe do pacote especial `java.util.concurrent` (como o `CopyOnWriteArrayList` ou `ConcurrentHashMap`).
</details>

**8. O Produtor-Consumidor (Padrão Antigo):** Antes de bibliotecas modernas, como era a "dança" primitiva (comunicação inter-thread) que os programadores faziam na mão para que o Produtor pausasse se o estoque lotasse, e o Consumidor pausasse se o estoque zerasse?
<details>
<summary>👀 Ver Resposta</summary>

Usava-se uma estrutura absurdamente complexa e frágil invocando, de forma manual e combinada com blocos `synchronized`, os métodos `.wait()` (mandava a thread dormir na marra) e o `.notify()` / `.notifyAll()` (dava um grito no sistema para acordar as outras threads informando que houve alteração no estoque).
</details>

**9. O Produtor-Consumidor Moderno:** Qual é o nome da estrutura de dados mágica introduzida no pacote *concurrent* que absorveu toda a lógica brutal do `wait/notify` do passado, automatizando a dormência de Produtores e Consumidores em estoques?
<details>
<summary>👀 Ver Resposta</summary>

É a estrutura `BlockingQueue` (como sua implementação mais famosa, a `ArrayBlockingQueue`). O método dela `.put()` e `.take()` já gerenciam as amarras de Lock, dormência e despertamento das threads invisivelmente e à prova de falhas humanas.
</details>

**10. Os Piscinões de Threads (ExecutorService):** No mundo comercial real (servidores), não usamos dezenas de 
ew Thread().start()`. Usamos **Thread Pools**. Explique o conceito de Thread Pool e cite por que o método `.shutdown()` de um Executor é crítico ao final do processo.
<details>
<summary>👀 Ver Resposta</summary>

Thread Pools (como as do `ExecutorService`) são reservatórios que mantêm um número limite fixo de threads já prontas e recicladas em stand-by, onde nós apenas arremessamos "Tarefas" na fila e elas as devoram sem estourar a memória RAM do computador criando threads infinitas. O `.shutdown()` é crítico pois, se não o chamarmos, as threads trabalhadoras do Executor ficarão vivas em *idle* na memória eternamente esperando por mais tarefas, impedindo que a aplicação Java inteira feche/desligue, criando um processo Zumbi.
</details>




