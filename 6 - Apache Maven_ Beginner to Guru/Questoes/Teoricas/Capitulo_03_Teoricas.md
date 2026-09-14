# Questões Teóricas - Capítulo 03 (Compiling Java)

**1. javac vs java:** Qual é a função primária do comando `javac` em comparação ao comando `java` no ecossistema JDK? O que cada um recebe como entrada e o que produz como saída?
<details>
<summary>👀 Ver Resposta</summary>

O `javac` é o compilador Java; ele recebe arquivos de código-fonte legíveis por humanos (`.java`) e os transforma em bytecode portátil (`.class`). O comando `java` é o inicializador da Java Virtual Machine (JVM); ele recebe o bytecode compilado (`.class` ou `.jar`), carrega as classes na memória, interpreta ou compila via JIT para instruções de máquina nativas e executa o método `main`.
</details>

**2. O Papel do Bytecode e a Portabilidade:** Por que o Java compila para *bytecode* em vez de compilar diretamente para código de máquina nativo da CPU do sistema operacional (como fazem C ou C++)? Qual é o lema histórico associado a isso?
<details>
<summary>👀 Ver Resposta</summary>

O bytecode funciona como uma linguagem intermediária universal e independente de arquitetura de hardware. Isso viabiliza o lema histórico *"Write Once, Run Anywhere"* (WORA): o mesmo arquivo `.class` compilado em um sistema Linux x86 pode ser executado sem recompilação em sistemas Windows ou macOS ARM, desde que cada plataforma possua sua respectiva JVM instalada para traduzir o bytecode para a máquina local.
</details>

**3. Anatomia de um Pacote e Diretórios:** Por que a declaração de pacote no topo de um arquivo fonte (ex: `package com.empresa.util;`) obriga uma estrutura idêntica de pastas no sistema de arquivos? O que ocorre ao compilar se houver divergência?
<details>
<summary>👀 Ver Resposta</summary>

O Java utiliza o nome do pacote como identificador único de namespace para evitar colisões de classes e mapear a hierarquia física no sistema de arquivos. O compilador `javac` e o Classloader da JVM esperam estritamente que a classe `com.empresa.util.Calculadora` esteja localizada no caminho físico de pastas `com/empresa/util/Calculadora.class`. Se houver divergência entre o diretório físico e a declaração de `package`, a JVM lançará erros como `NoClassDefFoundError` ou falhas de resolução de símbolos.
</details>

**4. O Parâmetro Classpath (`-cp` / `-classpath`):** Qual é a finalidade do Classpath no Java e por que ele é indispensável durante a compilação ou execução de projetos que utilizam bibliotecas de terceiros?
<details>
<summary>👀 Ver Resposta</summary>

O Classpath instrui a JVM e o compilador `javac` sobre onde procurar classes (`.class`) e arquivos compactados (`.jar`) adicionais que não fazem parte da biblioteca padrão do Java Runtime. Se uma classe depende de uma biblioteca externa e o Classpath não apontar explicitamente para o diretório ou `.jar` correspondente, o compilador emitirá erro de compilação indicando que o símbolo não foi encontrado.
</details>

**5. Separadores de Caminho Multiplataforma:** Qual é a diferença crucial de separadores de caminho no Classpath entre sistemas operacionais Unix (Linux/macOS) e Windows? Por que isso é uma fonte clássica de problemas em builds manuais?
<details>
<summary>👀 Ver Resposta</summary>

Em sistemas baseados em Unix (Linux e macOS), múltiplos caminhos no Classpath são separados por dois-pontos (`:`), por exemplo: `-cp lib1.jar:lib2.jar:.`. No ambiente Windows, o caractere dois-pontos é reservado para letras de unidade de disco (como `C:\`), de modo que o separador obrigatório de Classpath é o ponto-e-vírgula (`;`), ex: `-cp lib1.jar;lib2.jar;.`. Scripts de compilação manuais que ignoram essa diferença quebram imediatamente ao mudar de sistema operacional.
</details>

**6. Anatomia de um Arquivo JAR:** O que é estruturalmente um arquivo `.jar` (*Java Archive*)? Qual utilitário do sistema operacional pode descompactá-lo e qual arquivo especial dentro dele define qual classe contém o método `main`?
<details>
<summary>👀 Ver Resposta</summary>

Estruturalmente, um arquivo `.jar` é um arquivo compactado padrão no formato **ZIP**, contendo classes compiladas (`.class`) e arquivos de recursos (imagens, propriedades, XMLs). Ele pode ser aberto e descompactado por qualquer utilitário ZIP comum (como `unzip` ou 7-Zip). O arquivo especial que define a classe de entrada principal da aplicação executável é o manifesto `META-INF/MANIFEST.MF`, através do cabeçalho `Main-Class: com.empresa.App`.
</details>

**7. O Conceito de "JAR Hell":** O que a comunidade Java convencionou chamar de *"JAR Hell"* no contexto do gerenciamento manual de dependências via linha de comando?
<details>
<summary>👀 Ver Resposta</summary>

*"JAR Hell"* refere-se ao estado de caos e fragilidade em projetos com muitas dependências manuais, onde ocorrem: conflitos de versões de uma mesma biblioteca no classpath (onde a primeira encontrada mascara as demais), ausência de dependências transitivas necessárias gerando `ClassNotFoundException` ou `NoClassDefFoundError` em tempo de execução, e sobreposições acidentais de nomes de pacotes em JARs distintos.
</details>

**8. O Sinalizador `-d` no `javac`:** Qual é a função do argumento `-d` durante a invocação do `javac` e por que utilizá-lo é considerado uma boa prática de organização de projetos?
<details>
<summary>👀 Ver Resposta</summary>

O argumento `-d <diretório>` especifica o diretório de destino onde os arquivos `.class` gerados devem ser gravados, garantindo que o compilador crie automaticamente toda a estrutura de subpastas correspondente aos pacotes. Seu uso é fundamental para manter a separação limpa entre código-fonte original (`src`) e artefatos binários compilados (`bin` ou `target`), evitando que arquivos `.class` poluam as pastas de desenvolvimento.
</details>

**9. Execução de Arquivo Único (JEP 330):** A partir do Java 11, o que a funcionalidade introduzida pela JEP 330 permite fazer diretamente com arquivos `.java` no terminal, e qual comando de compilação deixa de ser necessário previamente?
<details>
<summary>👀 Ver Resposta</summary>

A JEP 330 permite executar programas Java contidos em um único arquivo-fonte diretamente com o comando `java MeuPrograma.java`, sem a necessidade de executar previamente o `javac` manual para gerar um `.class` em disco. A JVM compila o código em memória e o executa instantaneamente, sendo ideal para scripts ágeis, testes rápidos e aprendizado.
</details>

**10. Por que Ferramentas de Build (como Maven) se Tornaram Essenciais:** Diante da complexidade de compilar manualmente via `javac`, listar dezenas de bibliotecas no Classpath e lidar com diferenças de SO, qual é o papel das ferramentas automatizadas de build como o Apache Maven?
<details>
<summary>👀 Ver Resposta</summary>

O Maven padroniza a estrutura de diretórios do projeto, resolve e faz o download automático de dependências transitivas da internet, gerencia o Classpath dinamicamente para cada sistema operacional, orquestra fases de compilação, execução de testes unitários e empacotamento em JAR, eliminando scripts manuais frágeis e garantindo que o build seja totalmente repetível em qualquer máquina ou esteira de CI/CD.
</details>
