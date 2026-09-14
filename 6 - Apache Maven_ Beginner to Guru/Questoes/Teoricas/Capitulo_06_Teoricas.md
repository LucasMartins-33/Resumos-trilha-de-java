# Questões Teóricas - Capítulo 06 (Common Maven Plugins)

**1. O Plugin `maven-clean-plugin`:** Qual é a fase padrão associada ao objetivo `clean:clean` e por que executar uma limpeza regular é crucial antes de gerar versões de release ou rodar suítes de testes complexas?
<details>
<summary>👀 Ver Resposta</summary>

O objetivo `clean:clean` é acionado na fase `clean`. Ele é crucial porque remove completamente a pasta `target/`. Sem a limpeza, classes compiladas deletadas do código-fonte, recursos obsoletos ou relatórios antigos de testes passados podem permanecer em `target/classes` ou `target/surefire-reports`, causando falhas de compilação fantasma, falsos positivos em testes ou inclusão acidental de arquivos indesejados no JAR empacotado.
</details>

**2. O Plugin `maven-compiler-plugin`:** Quais são as duas fases do ciclo de vida que acionam o plugin de compilação do Maven e quais diretórios de saída recebem os arquivos `.class` correspondentes?
<details>
<summary>👀 Ver Resposta</summary>

1. Fase `compile`: Executa o goal `compiler:compile`, compilando o código de `src/main/java` e gravando os bytecodes na pasta `target/classes`.
2. Fase `test-compile`: Executa o goal `compiler:testCompile`, compilando os testes unitários de `src/test/java` e gravando na pasta `target/test-classes`.
</details>

**3. O Plugin `maven-resources-plugin` e Filtragem de Recursos:** O que é o recurso de *Resource Filtering* (`<filtering>true</filtering>`) gerenciado pelo `maven-resources-plugin`? Dê um exemplo de placeholder resolvido por ele.
<details>
<summary>👀 Ver Resposta</summary>

Filtragem de recursos é o mecanismo pelo qual o Maven lê arquivos de configuração estáticos (como `application.properties` ou `app.yml`) em `src/main/resources` e substitui dinamicamente placeholders de variáveis do POM durante a cópia para `target/classes`. Por exemplo, a instrução `app.versao=${project.version}` será convertida automaticamente no valor real da versão do POM, como `app.versao=1.0.0`.
</details>

**4. `maven-surefire-plugin` vs Testes:** Qual convenção de nomenclatura de arquivos o `maven-surefire-plugin` adota por padrão para identificar e executar classes de teste unitário durante a fase `test`?
<details>
<summary>👀 Ver Resposta</summary>

Por padrão, o Surefire executa qualquer classe Java localizada sob `src/test/java` que atenda a um dos seguintes padrões de nome:
* `**/Test*.java` (classes iniciando com "Test")
* `**/*Test.java` (classes terminando com "Test")
* `**/*Tests.java` (classes terminando com "Tests")
* `**/*TestCase.java` (classes terminando com "TestCase")
</details>

**5. Abortar Build em Falhas de Teste:** O que acontece por padrão quando um teste unitário falha durante a execução do `mvn package`? Como é possível instruir o Maven a gerar o pacote ignorando a execução ou os erros de teste via linha de comando?
<details>
<summary>👀 Ver Resposta</summary>

Por padrão, o Maven interrompe imediatamente o build com erro (`BUILD FAILURE`), impedindo que a fase `package` empacote um artefato quebrado. Para forçar o empacotamento ignorando a execução de testes, pode-se passar o argumento `-DskipTests` (que compila os testes mas pula a execução) ou `-Dmaven.test.skip=true` (que ignora inclusive a compilação das classes de teste).
</details>

**6. O Plugin `maven-jar-plugin` e Executabilidade:** Por que um arquivo JAR gerado pelo `maven-jar-plugin` tradicionalmente não pode ser executado diretamente com `java -jar meu-app.jar` a menos que seja configurado? O que precisa ser adicionado ao seu arquivo de manifesto?
<details>
<summary>👀 Ver Resposta</summary>

Por padrão, o `maven-jar-plugin` apenas compacta as classes em um arquivo ZIP sem informar à JVM qual é a classe principal de inicialização. Para torná-lo executável via `java -jar`, deve-se configurar a seção `<archive><manifest>` do plugin no `pom.xml`, especificando a tag `<mainClass>com.empresa.MinhaClassePrincipal</mainClass>`, que injetará o atributo correspondente no arquivo `META-INF/MANIFEST.MF`.
</details>

**7. O Plugin `maven-failsafe-plugin` e Testes de Integração:** Por que o Maven utiliza um plugin separado (`maven-failsafe-plugin`) para testes de integração na fase `integration-test` e `verify`, em vez de usar o Surefire?
<details>
<summary>👀 Ver Resposta</summary>

Testes de integração frequentemente inicializam recursos externos caros (bancos de dados em contêineres, servidores embutidos). Se um teste de integração falhasse no Surefire, o build abortaria imediatamente sem permitir que a fase subsequente (`post-integration-test`) finalizasse ou limpasse esses recursos. O Failsafe separa a execução dos testes da checagem de erros: ele executa os testes na fase `integration-test`, permite que a fase `post-integration-test` desligue os servidores/contêineres, e apenas na fase `verify` ele analisa os resultados e falha o build caso algum teste tenha quebrado.
</details>

**8. O Plugin `maven-shade-plugin` e "Fat JARs":** Qual é a finalidade primária do `maven-shade-plugin` e como ele resolve o problema de distribuir aplicações Java independentes? O que é o processo de *Relocation*?
<details>
<summary>👀 Ver Resposta</summary>

O `maven-shade-plugin` empacota a aplicação e todas as suas dependências externas em um único arquivo JAR autossuficiente (*Fat JAR* ou *Uber JAR*), descompactando os `.class` de todas as bibliotecas e mesclando-os com as classes do projeto. O recurso de **Relocation** permite alterar o nome do pacote de dependências de terceiros conflitantes (ex: remapear `org.objectweb.asm` para `minha.empresa.shaded.asm`), evitando colisões de classes em ambientes onde o Classpath já possui versões diferentes da mesma biblioteca.
</details>

**9. O Plugin `maven-source-plugin`:** Por que projetos de código aberto ou bibliotecas corporativas compartilhadas configuram o `maven-source-plugin` para ser executado no ciclo de compilação?
<details>
<summary>👀 Ver Resposta</summary>

Ele empacota o código-fonte original em um arquivo paralelo com o classificador `-sources.jar` (ex: `minha-lib-1.0.0-sources.jar`). Ao publicar esse arquivo junto ao JAR binário em repositórios remotos (como Maven Central ou Nexus), permite que desenvolvedores consumindo a biblioteca possam inspecionar o código-fonte original e depurar (*debugging*) métodos diretamente dentro de suas IDEs (IntelliJ/Eclipse).
</details>

**10. O Plugin `maven-deploy-plugin`:** Qual é a responsabilidade do objetivo `deploy:deploy` e quais elementos do `pom.xml` e do `settings.xml` são obrigatórios para que ele funcione sem erros?
<details>
<summary>👀 Ver Resposta</summary>

O `maven-deploy-plugin` copia o artefato empacotado final, seu respectivo arquivo `.pom` e metadados para um repositório remoto compartilhado (Nexus, Artifactory, Packagecloud ou Maven Central). Para funcionar, exige o bloco `<distributionManagement>` no `pom.xml` (especificando as URLs dos repositórios de release e snapshot) e a seção `<servers><server>` no arquivo `settings.xml` (fornecendo o usuário e senha correspondentes ao `<id>` do repositório para autenticação).
</details>
