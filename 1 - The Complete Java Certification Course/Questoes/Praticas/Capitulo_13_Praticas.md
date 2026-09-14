# Questões Práticas - Capítulo 13 (Advanced Concepts & JAR Files)

🟢 Nível 1: A Interface Marcadora e o Clone
Cenário: Em um jogo estilo MOBA, os lacaios (Minions) nascem aos milhares. Não queremos gastar memória dando `new` a cada instante, queremos apenas clonar o minion original.
Sua Tarefa: 
* Crie a classe `Minion`.
* Modifique a assinatura da classe para permitir a clonagem oficial do Java (Interface Marcadora).
* Apenas implemente o contrato da interface (ela é vazia).

🟡 Nível 2: O Método de Clonagem (Sobrescrita)
Cenário: Agora que a classe tem permissão, precisamos abrir as portas do método `clone()`.
Sua Tarefa:
* Na classe `Minion`, digite o método `@Override public Object clone()`.
* Na assinatura do método, adicione o alerta de exceção: `throws CloneNotSupportedException`.
* No corpo, retorne simplesmente a chamada para o construtor pai: `return super.clone();`.

🟠 Nível 3: Executando o Clone no Main
Cenário: O minion original precisa ser copiado!
Sua Tarefa:
* No `main`, crie o original: `Minion m1 = new Minion();`.
* Tente cloná-lo: `Minion m2 = m1.clone();`. Você vai se deparar com dois erros da IDE.
* Primeiro erro: Trate ou lance o `CloneNotSupportedException` no main (coloque `throws` na assinatura do main).
* Segundo erro: O `.clone()` devolve algo do tipo genérico `Object`. Você precisará fazer o *casting* explícito colocando `(Minion)` antes do método. 

🔴 Nível 4: O Contrato de Ordem (Comparable)
Cenário: Você tem uma lista de `Carro` (onde o atributo principal é `int velocidadeMaxima`). Ao invocar `Collections.sort(lista)`, o código quebra.
Sua Tarefa:
* Modifique a assinatura da classe `Carro` implementando a interface `Comparable<Carro>`.
* Crie na classe o método exigido: `public int compareTo(Carro outroCarro)`.

🟣 Nível 5: O Código da Ordem Natural
Cenário: Continuamos no `compareTo` do `Carro`. Queremos ordem decrescente (o mais rápido vem primeiro).
Sua Tarefa:
* Dentro do `compareTo`, crie uma árvore de IF/ELSE:
* Se `this.velocidadeMaxima == outroCarro.velocidadeMaxima`, retorne `0`.
* Se `this.velocidadeMaxima < outroCarro.velocidadeMaxima`, queremos que o carro atual vá para trás. Então, retorne `-1` (ou 1, dependendo da ordem que a questão pede — para carros velozes *primeiro*, se o atual é lento ele deve "perder" espaço).
* Pratique a lógica para garantir o retorno 1 ou -1.

🟤 Nível 6: O Marcador da Serialização
Cenário: Precisamos salvar o estado atual do `Carro` no disco rígido.
Sua Tarefa:
* Adicione à classe `Carro` a permissão para virar bytes, implementando a interface correta (`java.io.Serializable`).

🔵 Nível 7: O Campo Temporário (transient)
Cenário: O `Carro` agora tem um atributo `String chaveSecretaIgnicao`. Se ele for salvo no HD, a chave vazará para hackers.
Sua Tarefa:
* Adicione o modificador/palavra-chave necessário na declaração dessa variável para que o mecanismo de Serialização do Java a ignore durante o salvamento.

🟢 Nível 8: Comandos de Compilação
Cenário: Você terminou de escrever o arquivo `Main.java` e o servidor Linux de produção não tem Eclipse.
Sua Tarefa: (Apenas em formato de texto/comentário)
* Escreva o comando de terminal necessário para compilar esse arquivo.
* Escreva o comando subsequente para rodar o arquivo compilado na JVM.

🟡 Nível 9: O Manifest
Cenário: O projeto cresceu e tem 50 arquivos `.class`. Precisamos gerar o `manifest.mf`.
Sua Tarefa: (Em formato de texto)
* Escreva as duas linhas de texto puras exigidas dentro de um `manifest.mf` para informar que a classe inicial se chama `Application`. (Dica: lembre-se do Enter!).

🟠 Nível 10: Criando o JAR
Cenário: Todos os `.class` e o `manifest.mf` estão na pasta atual.
Sua Tarefa: (Em formato de texto/comentário)
* Escreva o comando completo que utiliza o programa de `jar` com as flags de criar, injetar o manifesto, e nomear o arquivo de saída como `MeuApp.jar`.
