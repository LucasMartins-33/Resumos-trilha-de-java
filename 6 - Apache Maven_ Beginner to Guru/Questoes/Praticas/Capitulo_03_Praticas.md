# Questões Práticas - Capítulo 03 (Compiling Java)

🟢 Nível 1: Sua Primeira Compilação Manual (`javac`)
Cenário: Você está sem IDE e precisa compilar um arquivo Java puro no terminal para entender o que as ferramentas de build fazem por baixo dos panos.
Sua Tarefa:
* Crie um arquivo `HelloWorld.java` com uma classe pública e o método `main` imprimindo `"Olá do javac!"`.
* Abra o terminal e execute o compilador Java: `javac HelloWorld.java`.
* Verifique com `ls` ou `dir` se o arquivo `HelloWorld.class` foi gerado no mesmo diretório.

🟡 Nível 2: Executando o Bytecode com a JVM (`java`)
Cenário: Com o arquivo `HelloWorld.class` gerado no Nível 1, você agora precisa pedir para a máquina virtual Java executá-lo.
Sua Tarefa:
* Execute a classe compilada no terminal usando o comando `java HelloWorld` (sem a extensão `.class`).
* Observe a saída do console.
* Teste o erro proposital: digite `java HelloWorld.class` e analise o erro `ClassNotFoundException` que a JVM retorna para entender por que a extensão nunca deve ser passada.

🟠 Nível 3: Organizando com Pacotes e a Flag `-d`
Cenário: Deixar arquivos `.class` misturados no mesmo diretório dos fontes `.java` gera desorganização. Você quer que a saída vá para uma pasta `bin/`.
Sua Tarefa:
* Adicione a instrução de pacote `package com.curso.maven;` na primeira linha do `HelloWorld.java`.
* Execute o comando `javac -d bin HelloWorld.java`.
* Inspecione a pasta `bin/` e observe como o `javac` criou automaticamente a árvore de diretórios `bin/com/curso/maven/HelloWorld.class`.

🔴 Nível 4: Executando Classes com Pacote e Classpath (`-cp`)
Cenário: Tentando rodar o arquivo do Nível 3 direto da pasta `bin/com/curso/maven` com `java HelloWorld`, a JVM falha com erro de pacote.
Sua Tarefa:
* Posicione seu terminal na raiz (fora da pasta `bin`).
* Execute a classe informando seu nome totalmente qualificado (FQN) e apontando o Classpath para a pasta `bin`:
  `java -cp bin com.curso.maven.HelloWorld`.
* Garanta que a mensagem seja exibida com sucesso.

🟣 Nível 5: Compilando Múltiplos Arquivos Dependentes
Cenário: Sua aplicação agora é composta por duas classes: `MensagemService.java` (que retorna um texto) e `App.java` (que chama o serviço no `main`).
Sua Tarefa:
* Coloque ambas no pacote `com.curso.maven`.
* Compile as duas de uma só vez usando o curinga: `javac -d bin *.java` (ou listando ambos os arquivos).
* Execute a classe `com.curso.maven.App` a partir da pasta `bin` para testar a comunicação entre as classes.

🟤 Nível 6: Integrando uma Biblioteca Externa Manualmente
Cenário: Você baixou um arquivo `commons-lang3-3.12.0.jar` da internet e quer usar `StringUtils.capitalize("maven")` na sua classe `App.java`.
Sua Tarefa:
* Tente compilar com `javac -d bin App.java` e veja o erro de compilação por falta do símbolo `StringUtils`.
* Adicione o JAR ao Classpath de compilação: `javac -cp "commons-lang3-3.12.0.jar" -d bin App.java`.
* Execute a aplicação no terminal passando ambos os caminhos no Classpath: `java -cp "bin:commons-lang3-3.12.0.jar" com.curso.maven.App` (ou `;` se estiver no Windows).

🔵 Nível 7: Empacotando em um Arquivo JAR
Cenário: Você precisa distribuir sua aplicação compilada para outro desenvolvedor como um arquivo único compactado.
Sua Tarefa:
* Navegue até a pasta `bin/`.
* Crie um arquivo JAR chamado `meu-app.jar` contendo todas as classes compiladas:
  `jar -cvf meu-app.jar com/curso/maven/*.class`.
* Inspecione o conteúdo do JAR gerado usando o comando `jar -tf meu-app.jar`.

🟢 Nível 8: Criando um JAR Executável com Manifesto
Cenário: Ao tentar rodar `java -jar meu-app.jar`, a JVM reclama: `"no main manifest attribute"`.
Sua Tarefa:
* Crie um arquivo de texto simples chamado `manifest.txt` com o conteúdo:
  `Main-Class: com.curso.maven.App` (lembre-se de dar um ENTER após a linha).
* Empacote o JAR novamente incluindo o manifesto:
  `jar -cvfm meu-app.jar manifest.txt com/curso/maven/*.class`.
* Execute diretamente com `java -jar meu-app.jar` e comprove a execução sem precisar especificar o nome da classe.

🟡 Nível 9: Diagnóstico de "Classpath Hell" Multiplataforma
Cenário: Um colega do Windows enviou um script de build que contém `-cp bin;libs/log.jar`, mas ao rodar no Linux a execução falha acusando sintaxe inválida ou classe não encontrada.
Sua Tarefa:
* Identifique a causa raiz: separador `;` do Windows vs `:` do Unix/Linux.
* Refatore a linha de comando no Linux para utilizar `:`: `-cp "bin:libs/log.jar"`.
* Reflita sobre a fragilidade de manter scripts shell `.sh` e `.bat` manuais para projetos grandes.

🟠 Nível 10: Execução Direta sem Compilação Prévia (JEP 330)
Cenário: Você precisa criar um script rápido em Java 11+ para processar um arquivo e não quer ter que rodar `javac` e depois `java`.
Sua Tarefa:
* Crie um arquivo `ScriptRapido.java` com método `main`.
* No terminal, execute diretamente: `java ScriptRapido.java`.
* Verifique com `ls` que **nenhum** arquivo `.class` foi gerado no disco, pois a compilação ocorreu em memória.
