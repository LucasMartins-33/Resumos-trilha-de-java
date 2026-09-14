# Capítulo 22: Concurrency in Java (Multithreading)

Este capítulo introduz um dos tópicos mais complexos e poderosos do Java: Multithreading (Programação Concorrente). Em vez de rodar o programa inteiro em uma única "linha do tempo" sequencial (Main Thread), podemos dividir tarefas para rodarem simultaneamente.

## 1. Criando e Rodando Threads

Existem duas formas principais de criar uma Thread no Java (tarefas independentes):
*   **Extendendo a classe `Thread`:** `public class Task extends Thread { public void run() { ... } }`
*   **Implementando a interface `Runnable`:** `public class Task implements Runnable { public void run() { ... } }` (Geralmente preferível, pois permite estender outras classes).

**Regra de Ouro:** Para iniciar uma Thread, **NUNCA chame o método `.run()` diretamente.** Você deve instanciar a Thread e chamar o método **`.start()`**.
*Se você chamar `.run()`, o código vai rodar de forma sequencial, travando a Main Thread. O `.start()` avisa a JVM para spawnar a nova Thread e executar o método `run()` paralelamente.*

```java
// Criando usando Runnable
Runnable tarefa = new Task();
Thread t1 = new Thread(tarefa);
t1.setName("Thread-A"); // Renomeando a thread
t1.start(); // Inicia paralelamente!
```

**Dicas úteis:**
*   `Thread.sleep(1000)`: Pausa a thread atual por X milissegundos.
*   `Thread.currentThread().getName()`: Retorna o nome da thread rodando aquela linha de código.
*   `t1.join()`: Faz a thread atual (ex: Main) pausar e esperar a `t1` terminar antes de prosseguir.

## 2. Thread Safety e Sincronização

Quando múltiplas threads acessam e tentam modificar os mesmos dados ao mesmo tempo, ocorre o "Interleaving", gerando resultados bizarros (ex: dois trabalhadores atualizando o número "3" para "4" ao mesmo tempo, perdendo o "5").

Para garantir **Atomicidade** (ou a linha roda inteira sem interrupção, ou não roda), usamos a palavra-chave **`synchronized`**.

```java
// Opção 1: Método inteiro sincronizado
public synchronized int getNext() { ... }

// Opção 2: Bloco Sincronizado (melhor performance, tranca só o necessário)
public void processData() {
    synchronized(this) { 
        // O(A) Thread que entrar aqui cria um "Lock" no objeto 'this'.
        // Nenhuma outra Thread acessa esse bloco até a primeira sair.
    }
}
```

### Coleções Concorrentes
Listas comuns (`ArrayList`, `HashMap`) NÃO SÃO "Thread Safe". Se uma thread adicionar itens na lista enquanto outra tenta lê-la, o Java jogará uma `ConcurrentModificationException`.
*   **Solução:** Use o pacote `java.util.concurrent`.
*   Exemplo: Use `CopyOnWriteArrayList` no lugar de `ArrayList`.

## 3. Padrão Producer-Consumer (Produtor-Consumidor)

Padrão onipresente em que uma Tarefa produz dados (Ex: Pega perguntas) e outra Tarefa consome dados (Ex: Responde as perguntas), ambas dividindo um estoque central.

*   **A abordagem Antiga e Complexa (`wait` e `notify`):** Exigia usar o `synchronized`, checar limites, chamar `.wait()` para dormir se o estoque estivesse cheio/vazio, e `.notify()` para acordar as outras threads. Muito propenso a erros.
*   **A abordagem Moderna (`ArrayBlockingQueue`):** Usa-se as coleções nativas do pacote concurrent. A `BlockingQueue` gerencia sozinha o limite (capacity). Você apenas usa `.put(item)` (produtor insere e dorme automático se lotar) e `.take()` (consumidor retira e dorme automático se zerar). Simples e 100% *Thread Safe* sem precisar usar `synchronized` manual.

## 4. O Framework Executor e Thread Pools

No mundo real, não ficamos criando instâncias de `.start()` manualmente. É difícil de gerenciar. Usamos **Thread Pools** (Piscinas de Threads).

O `ExecutorService` permite que você crie um limite de Threads ativas simultaneamente. Você submete 1000 "tarefas" para a piscina, e as X threads vão matando essas tarefas. Assim que uma Thread fica ociosa, ela já puxa a próxima tarefa da fila (Reciclagem de Threads).

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

// Cria uma piscina com apenas 2 threads trabalhadoras
ExecutorService executor = Executors.newFixedThreadPool(2);

// Envia dezenas de tarefas (Runnable). As 2 threads vão se dividir para processar tudo.
executor.execute(new MessageProcessor(1));
executor.execute(new MessageProcessor(2));
executor.execute(new MessageProcessor(3));

// REGRA CRÍTICA: Sempre lembre de fechar o Executor, senão a aplicação não morre (fica rodando no fundo esperando mais jobs).
executor.shutdown(); // Fecha a "porta", não aceita novas tarefas e desliga de forma suave quando os processos terminarem.
// executor.shutdownNow(); // Desligamento imediato e forçado (Corta no meio do processamento).
```
