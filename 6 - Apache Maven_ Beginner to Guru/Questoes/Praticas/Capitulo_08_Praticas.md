# Questões Práticas - Capítulo 08 (Alternate JVM Languages)

🟢 Nível 1: Estruturando Diretórios para Linguagens Alternativas
Cenário: Você vai criar um projeto poliglota que abrigará código Java e código Groovy convivendo pacificamente.
Sua Tarefa:
* Crie a estrutura de diretórios: `src/main/java` e `src/main/groovy`.
* Crie também a estrutura de testes: `src/test/java` e `src/test/groovy`.
* Crie uma classe Java simples em `src/main/java/com/minhaempresa/JavaService.java`.

🟡 Nível 2: Adicionando Dependências do Apache Groovy
Cenário: Para compilar e executar código Groovy na JVM, o Maven precisa incluir as bibliotecas de runtime do Groovy.
Sua Tarefa:
* Abra o `pom.xml` e adicione a dependência `org.apache.groovy:groovy:4.0.15` (ou Groovy 3.x).
* Crie um arquivo `src/main/groovy/com/minhaempresa/CalculadorGroovy.groovy` com uma classe Groovy simples que soma dois números.

🟠 Nível 3: Configurando o `gmavenplus-plugin`
Cenário: O Maven padrão não compila arquivos `.groovy`. Você precisa acoplar o plugin oficial GMavenPlus.
Sua Tarefa:
* No `pom.xml`, declare o plugin `org.codehaus.gmavenplus:gmavenplus-plugin:3.0.2`.
* Vincule os goals `addSources`, `addTestSources`, `compile` e `compileTests` às respectivas fases do ciclo de vida.
* Execute no terminal: `mvn clean compile`.
* Inspecione a pasta `target/classes/com/minhaempresa/` e comprove que o bytecode do `CalculadorGroovy.class` foi gerado normalmente.

🔴 Nível 4: Interoperabilidade Bidirecional Java ➔ Groovy
Cenário: Você precisa chamar a classe escrita em Groovy de dentro da sua classe Java.
Sua Tarefa:
* Na classe `JavaService.java`, instancie `new CalculadorGroovy()`.
* Chame o método de soma do Groovy e imprima o resultado.
* Execute `mvn compile` e verifique que o compilador conseguiu resolver a classe Groovy sem erros.

🟣 Nível 5: Testes Expressivos com Spock Framework
Cenário: Você quer adotar o Spock para escrever testes unitários BDD altamente legíveis no projeto.
Sua Tarefa:
* Adicione as dependências `org.spockframework:spock-core:2.4-M1-groovy-4.0` com escopo `test`.
* Crie a especificação `src/test/groovy/com/minhaempresa/CalculadorSpec.groovy` herdando de `spock.lang.Specification`.
* Escreva um teste com os blocos `given:`, `when:`, `then:`.
* Execute `mvn test` e observe o Surefire executando o teste Spock com sucesso.

🟤 Nível 6: Integrando Kotlin ao Projeto Maven
Cenário: Sua equipe quer começar a escrever novos microsserviços utilizando a linguagem Kotlin da JetBrains.
Sua Tarefa:
* Crie o diretório `src/main/kotlin`.
* Adicione a propriedade `<kotlin.version>1.9.22</kotlin.version>` no `pom.xml`.
* Adicione a dependência da biblioteca padrão `org.jetbrains.kotlin:kotlin-stdlib`.
* Crie o arquivo `src/main/kotlin/com/minhaempresa/Pessoa.kt` com a data class: `data class Pessoa(val id: Long, val nome: String)`.

🔵 Nível 7: Configurando o `kotlin-maven-plugin`
Cenário: Você precisa configurar o compilador de Kotlin para rodar de forma perfeitamente orquestrada com o compilador Java.
Sua Tarefa:
* No `pom.xml`, configure o plugin `org.jetbrains.kotlin:kotlin-maven-plugin`.
* Vincule o goal `compile` à fase `process-sources`.
* Vincule o goal `test-compile` à fase `process-test-sources`.
* Execute `mvn clean compile` e confirme a compilação do arquivo Kotlin.

🟢 Nível 8: Compilação Conjunta Kotlin e Java
Cenário: Você tem classes Java que consomem classes Kotlin e classes Kotlin que consomem classes Java no mesmo módulo.
Sua Tarefa:
* Na classe `JavaService.java`, instancie `new Pessoa(1L, "Maria")` e exiba o nome.
* No `pom.xml`, configure o `maven-compiler-plugin` para desativar a compilação padrão avulsa e rodar em conjunto após o Kotlin:
  adicione uma execução do `compile` vinculada à fase `compile`.
* Execute `mvn clean compile` e comprove a resolução mista de símbolos.

🟡 Nível 9: Configurando Scala com o `scala-maven-plugin`
Cenário: Você precisa compilar um componente analítico escrito em Scala.
Sua Tarefa:
* Crie o diretório `src/main/scala`.
* Adicione a dependência `org.scala-lang:scala-library:2.13.12`.
* Adicione o plugin `net.alchim31.maven:scala-maven-plugin:4.8.1` com os goals `compile` e `testCompile`.
* Crie um `Object` Scala simples em `src/main/scala/com/minhaempresa/CalculoScala.scala`.
* Execute `mvn compile` e verifique a geração do bytecode correspondente.

🟠 Nível 10: O Plugin `build-helper-maven-plugin` para IDEs
Cenário: O IntelliJ IDEA não reconheceu automaticamente a pasta `src/main/kotlin` como pasta de código-fonte após a importação.
Sua Tarefa:
* Adicione o plugin `org.codehaus.mojo:build-helper-maven-plugin:3.4.0` ao `pom.xml`.
* Configure uma execução com o goal `add-source` na fase `generate-sources` apontando para `<source>src/main/kotlin</source>`.
* Execute `mvn generate-sources` e atualize o projeto na IDE (Maven Reload).
* Observe a pasta `src/main/kotlin` ser automaticamente marcada com a cor azul de código-fonte na árvore da IDE.
