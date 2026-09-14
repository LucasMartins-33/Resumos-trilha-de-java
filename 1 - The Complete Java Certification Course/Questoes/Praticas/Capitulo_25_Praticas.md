# Questões Práticas - Capítulo 25 (Java JShell)

*OBS: Como este é um capítulo 100% focado no Terminal e sem IDE, as "tarefas práticas" aqui são na verdade "Os Comandos que você deveria bater no console" caso estivesse operando o JShell ativamente no Windows.*

🟢 Nível 1: Dando a Partida
Cenário: Você precisa testar comandos Java super rápidos, sem gastar 3 minutos para subir o Eclipse.
Sua Tarefa (Ação de Terminal):
* Abra o seu CMD (Windows) ou Terminal nativo (Linux/Mac) na sua máquina.
* Digite o gatilho `jshell` para imergir no ecosistema do Java Interativo.

🟡 Nível 2: Liberdade Imediata
Cenário: Quebrar correntes da Sintaxe Boilerplate.
Sua Tarefa:
* Com o JShell aberto (`jshell>`), digite exatamente e puramente o comando `System.out.println("Funciona sem Main!");`. Aperte Enter. Note que não deu erro pela falta de uma Classe invólucro. 

🟠 Nível 3: O Cálculador Invisível (Scratch Variable)
Cenário: Você precisa da Raiz Quadrada de um número gigantesco de relance, mas quer evitar o trabalho braçal.
Sua Tarefa:
* Digite diretamente uma chamada pura do pacote Math nativo do Java: `Math.sqrt(1024)`. 
* Sem amarrar o valor a um `double x =`, o terminal exibirá a resposta com um rótulo especial automático, por exemplo: `$1 ==> 32.0`. Guarde mentalmente esse `$1`.

🔴 Nível 4: Reciclando as Variáveis do Sistema
Cenário: O JShell alocou o 32 em uma caixinha automática.
Sua Tarefa:
* Aproveitando-se que a caixa `$1` agora existe globalmente na memória, digite `$1 + 18` e mande para a tela. Ele deve processar 50 e alocar no próximo index de scratch (`$2` ou similar).

🟣 Nível 5: O Criador de Funções Espontâneas
Cenário: Você não está preso a ações unitárias, podemos moldar escopos ricos.
Sua Tarefa:
* Construa a função na mosca! Digite (em uma linha ou quebrando em multi-linhas com chaves abertas se sua versão do terminal curtir):
* `int somarNumeros(int a, int b) { return a + b; }` e aperte Enter. 
* O ambiente emitirá o eco que o método foi devidamente injetado na cache de código.

🟤 Nível 6: Invocação Direta
Cenário: Você precisa usar o método moldado no Nível 5.
Sua Tarefa:
* Digite na linha nova apenas a chamada crua: `somarNumeros(15, 30)` e avalie. 

🔵 Nível 7: Consultando a Memória do Passado (`/vars`)
Cenário: Você foi ao banheiro e quando voltou, o console rolou para cima e você esqueceu as variáveis que criou há meia-hora.
Sua Tarefa:
* Digite o comando de auditoria de variáveis: `/vars`.
* Ele trará um relatório impresso provando todas as instâncias vivas (e os seus dados ocultos $x), algo excelente para debugar a memória sem imprimir manualmente.

🟢 Nível 8: Revisão do Molde (`/methods`)
Cenário: Você não sabe mais se o seu método `somarNumeros` usava `int` ou `long`.
Sua Tarefa:
* Digite `/methods`. Ele exibirá a lista e a assinatura técnica visual de todos os escopos funcionais declarados na sessão, poupando a necessidade de dar "scroll up" infinito no terminal.

🟡 Nível 9: O Lixo do Dev (`/list` ou `/history`)
Cenário: O JShell não gera arquivo `.java` físico no HD. Todo o seu trabalho morre quando a tela preta fecha. Você quer exportar (copiar) tudo o que digitou nas últimas duas horas.
Sua Tarefa:
* Puxe o livro contábil da sessão: digite `/list`.
* O JShell despejará na tela o script em formato de arquivo puríssimo englobando todas as declarações que ele está engolindo desde a inicialização. (Ideal para selecionar, dar "Ctrl+C" e passar pra um arquivo limpo na IDE depois da prototipagem).

🟠 Nível 10: Fugindo do Shell (`/exit`)
Cenário: A exploração terminou com êxito. O teste rápido Java provou o conceito da lógica de negócios, e agora você deve retornar à IDE nativa do Eclipse.
Sua Tarefa:
* Limpe e desvincule a instância do console batendo no terminal o comando definitivo de aborto seguro: `/exit`.
* Você passará instantaneamente da tela `jshell>` para a tela de pastas do computador. O experimento está concluído!
