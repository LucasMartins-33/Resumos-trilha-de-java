# Questões Teóricas - Capítulo 14 (File Processing and Exception Handling)

**1. O Conceito de Exceção:** O que é uma Exceção em Java e qual a principal diferença de experiência de software entre deixar que a exceção "estoure" livremente na tela do usuário vs tratá-la via código?
<details>
<summary>👀 Ver Resposta</summary>

Uma exceção é um erro (um fluxo não previsto) ocorrido em tempo de execução. Se não tratada, o programa entra em pânico, aborta o funcionamento e joga a famosa "tela vermelha/stacktrace" assustadora para o usuário final. Se tratada, capturamos esse evento, exibimos uma mensagem amigável e permitimos que o programa continue rodando com segurança.
</details>

**2. A Janela do Teclado:** Para que serve a classe nativa `Scanner` e como fazemos a "ponte" entre ela e os botões pressionados pelo usuário?
<details>
<summary>👀 Ver Resposta</summary>

O `Scanner` serve para interpretar dados de diferentes fluxos (Textos, Arquivos etc). Para conectá-lo ao teclado, instanciamos a classe passando o fluxo de entrada do sistema operacional como argumento no construtor: 
ew Scanner(System.in)`.
</details>

**3. O Bloco Garantido:** Qual é a utilidade específica do bloco `finally` acoplado em uma estrutura de `try/catch` tradicional?
<details>
<summary>👀 Ver Resposta</summary>

O bloco `finally` contém códigos que DEVEM ser executados obrigatoriamente, independentemente se o `try` deu certo ou se o `catch` foi acionado devido a um erro. Historicamente, servia para rodar as rotinas de limpeza, como o fechamento de portas/arquivos e desconexões do Banco de Dados para evitar vazamentos de recursos (Memory Leaks).
</details>

**4. A Burocracia do Passado:** Qual era a grande burocracia, considerada por muitos um erro de design do Java, ao se lidar com fechamento de arquivos (`.close()`) no bloco `finally` antes da versão 7?
<details>
<summary>👀 Ver Resposta</summary>

Como a própria ação de invocar o método de fechar arquivo (`.close()`) também corria o risco de gerar uma `IOException`, os desenvolvedores eram obrigados a colocar um NOVO bloco inteiro de `try/catch` **dentro** do próprio bloco `finally`, tornando a leitura do código gigantesca, feia e extremamente redundante.
</details>

**5. A Evolução Sintática:** Explique o funcionamento mecânico da estrutura `Try-with-Resources` introduzida no Java 7. De que forma ela eliminou a necessidade do bloco `finally` para fechar arquivos?
<details>
<summary>👀 Ver Resposta</summary>

Em vez de abrir o recurso antes do Try, você declara as variáveis do arquivo **dentro dos parênteses do próprio Try**: `try (FileReader f = new FileReader("...")) { ... }`. Ao fazer isso, o Java intercepta o fechamento das chaves do try e chama silenciosa e magicamente o método de fechamento para nós, apagando a necessidade de escrevermos um `finally` com chamadas manuais.
</details>

**6. A Magia por Trás dos Panos:** Para que o `Try-with-Resources` faça seu fecho mágico e automático, qual é a Interface nativa que a classe (como `FileReader` ou até nossa própria classe) precisa obrigatoriamente implementar?
<details>
<summary>👀 Ver Resposta</summary>

A interface `java.lang.AutoCloseable`. Ela exige apenas que a classe possua um método `close()`. Qualquer objeto de uma classe que assine esse contrato pode ser engolido pelo try-with-resources.
</details>

**7. Capturar vs Alertar:** Qual a diferença fundamental de arquitetura de software entre tratar um erro localmente (envelopando a linha num bloco `try/catch`) e alertar/propagar o erro (colocando `throws Exception` na assinatura do método)?
<details>
<summary>👀 Ver Resposta</summary>

Usar `try/catch` assume a responsabilidade de lidar com a "sujeira" ali mesmo, tomando uma ação (ex: exibir mensagem e usar arquivo backup). Usar `throws` na assinatura é "lavar as mãos" e terceirizar a responsabilidade. Você avisa que o método é perigoso, e OBRIGA quem for invocar o seu método no futuro a montar um `try/catch` por conta própria.
</details>

**8. As Exceções Verificadas (Checked Exceptions):** Se você tentar escrever a linha `FileReader f = new FileReader("banco.txt")` jogada solta em um método, o compilador do Java (linhas vermelhas) proibirá a execução. Por quê?
<details>
<summary>👀 Ver Resposta</summary>

Essa ação pertence ao grupo de *Checked Exceptions* (ex: `IOException`). O Java considera que conversar com o mundo externo (arquivos, redes, DB) é de Altíssimo Risco de falha. Como a culpa não é nossa caso o pendrive pife (força maior), o compilador nos "obriga por lei" a usar `try/catch` ou repassar com `throws`, caso contrário o projeto não compila.
</details>

**9. As Exceções Não-Verificadas:** O que é a temida `NullPointerException` e por que veteranos dizem que é uma péssima prática tentar capturá-la montando um `try/catch (NullPointerException)`?
<details>
<summary>👀 Ver Resposta</summary>

É o erro ativado quando você tenta extrair propriedades/métodos de uma variável de referência que está apontando para o vazio/nada (
ull`). Faz parte das *Unchecked Exceptions* (Erros de Lógica). É péssima prática usar `try/catch` para contê-las porque erros lógicos (como esquecer de inicializar objetos) devem ser consertados com lógica preventiva: uma simples checagem `if (variavel == null)` é infinitamente mais elegante, semitântica e barata do que acionar a complexa engrenagem da máquina de exceções do Java.
</details>

**10. O Perigo de Esquecer de Fechar:** Por que é tão perigoso esquecer de executar o método `.close()` de fluxos (como Scanner de arquivos pesados, conexões à internet ou de banco de dados)?
<details>
<summary>👀 Ver Resposta</summary>

Quando você abre esses recursos, o Sistema Operacional e o Java reservam e bloqueiam espaços severos na memória (Streams, Handlers) para manter as conexões vivas. Se você não os fechar ativamente, esses dutos continuarão consumindo memória invisivelmente (Resource Leak), o que fará o aplicativo ficar progressivamente mais lento até esgotar a RAM do servidor e travar (Crash).
</details>




