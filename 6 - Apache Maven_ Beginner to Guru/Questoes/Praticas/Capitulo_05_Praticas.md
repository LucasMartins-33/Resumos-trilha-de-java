# Questões Práticas - Capítulo 05 (Maven Basics)

🟢 Nível 1: Inspecionando o Repositório Local (`.m2`)
Cenário: Você quer entender como o Maven guarda os arquivos baixados da internet no seu disco local.
Sua Tarefa:
* Abra o terminal e navegue até `${user.home}/.m2/repository`.
* Procure pela pasta da dependência adicionada no capítulo anterior: `org/apache/commons/commons-lang3/3.12.0/`.
* Verifique com `ls` os arquivos lá presentes: o `.jar`, o arquivo `.pom` e os checksums `.sha1`.

🟡 Nível 2: Inspecionando o Effective POM
Cenário: Você configurou poucas linhas no seu `pom.xml`, mas o Maven sabe exatamente quais diretórios e plugins usar. Você quer ver as configurações completas herdadas.
Sua Tarefa:
* No diretório do seu projeto, execute o comando: `mvn help:effective-pom`.
* Localize na saída onde o repositório **central** (`repo.maven.apache.org`) está definido.
* Identifique a versão padrão do `maven-surefire-plugin` que veio herdada do Super POM.

🟠 Nível 3: Explorando o Escopo `test`
Cenário: Você precisa adicionar o framework JUnit Jupiter ao projeto, mas essa biblioteca não deve ser distribuída no JAR de produção.
Sua Tarefa:
* Abra o `pom.xml` e adicione a dependência `org.junit.jupiter:junit-jupiter-api:5.10.0`.
* Adicione a tag `<scope>test</scope>`.
* Crie uma classe simples em `src/test/java/com/minhaempresa/PrimeiroTest.java` com um teste anotado com `@Test`.
* Execute `mvn test` e observe a execução do teste.

🔴 Nível 4: Validando o Isolamento do Escopo `test`
Cenário: Você quer comprovar na prática que o escopo `test` impede que classes de teste ou bibliotecas de teste vazem para o código principal.
Sua Tarefa:
* Abra a classe `Main.java` em `src/main/java`.
* Tente importar `org.junit.jupiter.api.Assertions;` dentro da classe `Main`.
* Execute `mvn compile`.
* Analise a falha de compilação: o Maven recusa o build porque a dependência com escopo `test` não existe no Classpath de compilação de `src/main/java`. Remova a linha incorreta.

🟣 Nível 5: Simulando o Escopo `provided`
Cenário: Você está criando um componente web para rodar dentro do Tomcat, e a API de Servlets será fornecida pelo próprio servidor em runtime.
Sua Tarefa:
* Adicione a dependência `jakarta.servlet:jakarta.servlet-api:6.0.0` ao `pom.xml`.
* Configure o escopo como `<scope>provided</scope>`.
* Crie uma classe que implemente ou importe interfaces da Servlet API.
* Execute `mvn package` e abra o JAR gerado (como um arquivo zip). Comprove que o JAR da Servlet API **não** foi embutido no pacote.

🟤 Nível 6: Investigando Conflitos de Versão com `dependency:tree`
Cenário: Você adicionou duas bibliotecas no projeto e suspeita que elas estão trazendo versões conflitantes de uma dependência transitiva comum.
Sua Tarefa:
* Adicione no `pom.xml` dependências que tragam dependências transitivas (ex: `org.springframework:spring-context:5.3.30` e `com.fasterxml.jackson.core:jackson-databind:2.15.2`).
* Execute no terminal: `mvn dependency:tree`.
* Localize os galhos da árvore e identifique quais bibliotecas secundárias foram puxadas automaticamente.

🔵 Nível 7: Resolução de Conflito "Nearest Wins"
Cenário: Uma biblioteca traz transitivamente a versão `1.2` de um utilitário, mas você precisa urgentemente da versão `2.0` para corrigir um bug.
Sua Tarefa:
* Declare explicitamente no seu bloco `<dependencies>` a biblioteca transitiva com `<version>2.0</version>`.
* Execute `mvn dependency:tree` novamente.
* Comprove que a versão `2.0` venceu porque está declarada na raiz do projeto (distância 1), substituindo a versão transitiva (distância 2).

🟢 Nível 8: Excluindo Dependências Transitivas (`<exclusions>`)
Cenário: Uma dependência antiga do seu projeto está puxando acidentalmente o `commons-logging`, mas o seu projeto padronizou o uso de SLF4J e Logback.
Sua Tarefa:
* No bloco `<dependency>` da biblioteca antiga, abra a tag `<exclusions>`.
* Crie uma `<exclusion>` especificando `groupId: commons-logging` e `artifactId: commons-logging`.
* Execute `mvn dependency:tree` e certifique-se de que o `commons-logging` desapareceu da árvore do build.

🟡 Nível 9: Centralizando com `<dependencyManagement>`
Cenário: Você quer padronizar as versões das bibliotecas do ecossistema Jackson em um único ponto para facilitar futuras atualizações.
Sua Tarefa:
* Crie a seção `<dependencyManagement>` antes do bloco `<dependencies>`.
* Dentro dela, declare `jackson-databind` com a versão `2.16.0`.
* No bloco de `<dependencies>` principal, declare a dependência do `jackson-databind` **sem a tag `<version>`**.
* Execute `mvn compile` e comprove que o Maven herdou a versão definida no gerenciamento.

🟠 Nível 10: Importando um BOM com `<scope>import</scope>`
Cenário: Em vez de manter dezenas de versões manuais de bibliotecas Spring, você deseja importar o catálogo curado oficial (*Bill of Materials*).
Sua Tarefa:
* Dentro de `<dependencyManagement>`, adicione uma dependência para `org.springframework:spring-framework-bom:6.1.2`.
* Configure obrigatoriamente `<type>pom</type>` e `<scope>import</scope>`.
* No bloco `<dependencies>`, adicione `spring-core` e `spring-beans` sem tag `<version>`.
* Execute `mvn clean compile` e valide a resolução perfeita das dependências a partir do BOM importado.
