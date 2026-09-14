# Questões Práticas - Capítulo 06 (Common Maven Plugins)

🟢 Nível 1: Executando Limpeza Profunda com `clean`
Cenário: O diretório `target/` está acumulando arquivos de compilações anteriores e você deseja restaurar o projeto para o estado virgem.
Sua Tarefa:
* Abra o terminal na raiz do projeto e execute: `mvn clean`.
* Verifique com `ls` se a pasta `target/` foi completamente removida.
* Execute `mvn clean:clean` diretamente (invocando o goal avulso do plugin) e compreenda a equivalência com a fase `clean`.

🟡 Nível 2: Customizando o `maven-compiler-plugin`
Cenário: Você quer garantir que o compilador mostre avisos detalhados de código obsoleto (*deprecated*) durante a compilação.
Sua Tarefa:
* No `pom.xml`, abra a seção `<build><plugins>`.
* Declare o `maven-compiler-plugin` com a versão `3.11.0`.
* Dentro de `<configuration>`, adicione a tag `<showDeprecation>true</showDeprecation>` e `<showWarnings>true</showWarnings>`.
* Escreva um método usando uma classe obsoleta (ex: `new Date(year, month, date)`) e rode `mvn compile` para ver os avisos no terminal.

🟠 Nível 3: Filtragem de Recursos Dinâmica (`resources:resources`)
Cenário: Você quer que a versão do sistema definida no POM seja injetada automaticamente em um arquivo `app.properties` empacotado no JAR.
Sua Tarefa:
* Crie o arquivo `src/main/resources/app.properties` com a linha: `versao=${project.version}`.
* No `pom.xml`, configure o `<build><resources><resource>` apontando para `src/main/resources` com `<filtering>true</filtering>`.
* Execute `mvn process-resources`.
* Abra o arquivo gerado em `target/classes/app.properties` e verifique se `${project.version}` foi substituído pela versão real (ex: `1.0-SNAPSHOT`).

🔴 Nível 4: Executando Testes Específicos com o Surefire
Cenário: Sua suíte possui dezenas de testes, mas você alterou apenas a classe `Calculadora` e deseja rodar somente o teste correspondente para economizar tempo.
Sua Tarefa:
* Crie duas classes de teste em `src/test/java`: `CalculadoraTest.java` e `UsuarioTest.java`.
* No terminal, execute apenas o teste desejado passando o parâmetro `-Dtest`:
  `mvn test -Dtest=CalculadoraTest`.
* Verifique no log do console que o `UsuarioTest` foi completamente ignorado.

🟣 Nível 5: Pular Testes no Empacotamento
Cenário: Em um ambiente de emergência, você precisa gerar o pacote JAR rapidamente para testar a infraestrutura, sem esperar a execução demorada da suíte de testes.
Sua Tarefa:
* Execute o empacotamento com: `mvn package -DskipTests`.
* Observe no log do Maven que os testes foram compilados, mas o Surefire emitiu: `Tests are skipped`.
* Inspecione a pasta `target/` e confirme que o JAR foi gerado com sucesso.

🟤 Nível 6: Criando um JAR Executável com `maven-jar-plugin`
Cenário: Você quer que seu JAR seja inicializado no terminal com um simples comando `java -jar app.jar`.
Sua Tarefa:
* No `pom.xml`, adicione a configuração do `maven-jar-plugin`.
* Dentro de `<configuration><archive><manifest>`, configure `<mainClass>com.minhaempresa.Main</mainClass>`.
* Execute `mvn clean package`.
* Teste a execução direta: `java -jar target/primeiro-projeto-1.0-SNAPSHOT.jar`.

🔵 Nível 7: Configurando Testes de Integração com o Failsafe
Cenário: Você quer separar testes unitários rápidos de testes de integração mais lentos que dependem de recursos externos.
Sua Tarefa:
* Crie um teste chamado `src/test/java/com/minhaempresa/BancoDadosIT.java` seguindo a convenção `*IT.java`.
* Adicione o `maven-failsafe-plugin` ao `pom.xml` com as execuções para os goals `integration-test` e `verify`.
* Execute `mvn test` e observe que o `BancoDadosIT` **não** foi executado.
* Execute `mvn verify` e comprove que agora tanto os testes unitários quanto o teste de integração foram acionados.

🟢 Nível 8: Gerando um Fat JAR com o `maven-shade-plugin`
Cenário: Você precisa distribuir sua aplicação de linha de comando para usuários que não possuem o Maven nem bibliotecas instaladas na máquina.
Sua Tarefa:
* Adicione o `maven-shade-plugin` com a versão `3.5.0` na seção de plugins do `pom.xml`.
* Vincule o goal `shade` à fase `package`.
* Execute `mvn clean package`.
* Inspecione a pasta `target/`: observe que um arquivo `original-primeiro-projeto-...jar` foi criado (o JAR fino) e o JAR principal foi reempacotado contendo todas as classes de terceiros embutidas.

🟡 Nível 9: Anexando Código-Fonte com `maven-source-plugin`
Cenário: Você está desenvolvendo uma biblioteca compartilhada e deseja que os outros times consigam navegar no seu código-fonte através da IDE.
Sua Tarefa:
* Configure o `maven-source-plugin` no `pom.xml`.
* Vincule o goal `jar-no-fork` à fase `package` ou `verify`.
* Execute `mvn package`.
* Verifique na pasta `target/` a presença do arquivo adicional `primeiro-projeto-1.0-SNAPSHOT-sources.jar`.

🟠 Nível 10: Gerando a Documentação Javadoc com `maven-javadoc-plugin`
Cenário: O comitê de arquitetura exige que toda biblioteca tenha documentação HTML de suas APIs gerada no build.
Sua Tarefa:
* Adicione comentários Javadoc (`/** ... */`) nos métodos públicos da classe `Main`.
* Adicione o `maven-javadoc-plugin` no `pom.xml`.
* Execute no terminal: `mvn javadoc:javadoc`.
* Abra o arquivo `target/site/apidocs/index.html` em um navegador web e navegue pela documentação gerada da sua aplicação.
