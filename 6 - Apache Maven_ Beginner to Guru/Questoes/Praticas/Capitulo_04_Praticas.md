# Questões Práticas - Capítulo 04 (Getting Started with Maven)

🟢 Nível 1: Estruturando o Primeiro `pom.xml` Mínimo
Cenário: Você está iniciando um projeto do zero sem IDE e precisa criar manualmente o arquivo XML descritor que o Maven reconhece.
Sua Tarefa:
* Crie um arquivo `pom.xml` na raiz da pasta do projeto.
* Defina a tag raiz `<project>` com os namespaces padrão do Maven 4.0.0.
* Configure as coordenadas GAV mínimas: `groupId` (`com.minhaempresa`), `artifactId` (`primeiro-projeto`) e `version` (`1.0-SNAPSHOT`).

🟡 Nível 2: Criando o Standard Directory Layout
Cenário: O Maven falha ao compilar se a estrutura de pastas não seguir a convenção padrão.
Sua Tarefa:
* Crie a árvore de pastas padrão: `src/main/java` e `src/test/java`.
* Crie a pasta `src/main/resources`.
* Dentro de `src/main/java`, crie o pacote `com/minhaempresa` e adicione a classe `Main.java` com um `System.out.println("Maven em Ação!");`.

🟠 Nível 3: O Primeiro Build com `mvn clean compile`
Cenário: Com a estrutura e o código prontos, você deve compilar via terminal.
Sua Tarefa:
* No terminal, execute `mvn clean compile`.
* Analise a saída do console e localize a mensagem `BUILD SUCCESS`.
* Inspecione a pasta `target/classes/` gerada automaticamente e verifique se o arquivo `Main.class` foi criado no caminho do pacote correspondente.

🔴 Nível 4: Empacotando em JAR com `mvn package`
Cenário: Você precisa gerar o binário final da aplicação para distribuição.
Sua Tarefa:
* Execute no terminal o comando `mvn package`.
* Inspecione a pasta `target/` e localize o arquivo `primeiro-projeto-1.0-SNAPSHOT.jar`.
* Observe quais fases intermediárias o Maven executou no console antes de chegar no empacotamento (`validate`, `compile`, `test`).

🟣 Nível 5: Adicionando a Primeira Dependência Externa
Cenário: Você precisa utilizar o utilitário `commons-lang3` da Apache para manipular strings sem baixar arquivos JAR manualmente da web.
Sua Tarefa:
* Abra o `pom.xml` e crie o bloco `<dependencies>`.
* Adicione a dependência do `org.apache.commons:commons-lang3:3.12.0`.
* Execute `mvn compile` e observe no log o Maven fazendo o download do JAR a partir do Maven Central para o seu cache local `~/.m2/repository`.

🟤 Nível 6: Utilizando a Dependência no Código
Cenário: Com a dependência baixada pelo Maven, agora ela deve ser consumida no seu código.
Sua Tarefa:
* Abra a classe `Main.java`.
* Importe `org.apache.commons.lang3.StringUtils`.
* Imprima na tela o resultado de `StringUtils.reverse("Maven")`.
* Recompile o projeto com `mvn compile` para validar que o Classpath foi configurado automaticamente pelo Maven.

🔵 Nível 7: Configurando a Versão Moderna do Compilador Java
Cenário: Ao compilar, o Maven emite avisos de que está utilizando uma versão antiga do compilador (Java 5/7 padrão do super pom legado).
Sua Tarefa:
* No `pom.xml`, adicione a seção `<properties>`.
* Defina a propriedade moderna `<maven.compiler.release>17</maven.compiler.release>` (ou 11/21).
* Configure a codificação com `<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>`.
* Recompile com `mvn clean compile` e observe a ausência de avisos.

🟢 Nível 8: Protegendo o Repositório Git com `.gitignore`
Cenário: Você vai inicializar o repositório Git do projeto e não quer commitar arquivos temporários do Maven.
Sua Tarefa:
* Crie o arquivo `.gitignore` na raiz do projeto.
* Adicione a pasta `target/`.
* Adicione arquivos específicos de IDEs: `.idea/`, `*.iml`, `.settings/`, `.project`, `.classpath`.

🟡 Nível 9: Instalando e Usando o Maven Wrapper (`mvnw`)
Cenário: Você quer garantir que qualquer membro do time consiga compilar o projeto mesmo que não tenha o Apache Maven instalado no sistema operacional.
Sua Tarefa:
* Gere o wrapper no projeto executando no terminal: `mvn wrapper:wrapper`.
* Inspecione os arquivos gerados: `mvnw`, `mvnw.cmd` e a pasta `.mvn/wrapper/`.
* Teste a execução do build usando o wrapper: `./mvnw clean package` (Linux/Mac) ou `mvnw.cmd clean package` (Windows).

🟠 Nível 10: Importando e Executando no IntelliJ IDEA
Cenário: Você deseja trabalhar no projeto através da IDE gráfica IntelliJ IDEA.
Sua Tarefa:
* Abra o IntelliJ, selecione *Open* e aponte para o arquivo `pom.xml` da pasta.
* Abra a aba lateral direita do **Maven**.
* Navegue até *Lifecycle* e dê um duplo clique em `clean` e depois em `package`.
* Execute a classe `Main` diretamente pelo botão *Play* da IDE, conferindo a integração perfeita do POM com a IDE.
