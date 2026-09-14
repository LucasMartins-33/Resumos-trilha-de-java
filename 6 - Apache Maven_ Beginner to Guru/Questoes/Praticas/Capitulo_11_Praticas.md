# Questões Práticas - Capítulo 11 (Apache Maven for Spring Boot)

🟢 Nível 1: Criando uma Aplicação com `spring-boot-starter-parent`
Cenário: Você precisa configurar um projeto Spring Boot do zero usando Maven padrão.
Sua Tarefa:
* Crie o `pom.xml` definindo o bloco `<parent>` com o `org.springframework.boot:spring-boot-starter-parent:3.2.4`.
* No bloco `<dependencies>`, adicione o starter básico: `org.springframework.boot:spring-boot-starter-web` (sem informar a tag `<version>`).
* Crie a classe `DemoApplication.java` anotada com `@SpringBootApplication` e o método `main`.

🟡 Nível 2: Executando a Aplicação via Maven CLI (`spring-boot:run`)
Cenário: Você quer iniciar o servidor web embutido da aplicação diretamente pelo terminal sem abrir uma IDE.
Sua Tarefa:
* Abra o terminal na raiz do projeto e execute: `mvn spring-boot:run`.
* Observe o banner do Spring Boot e os logs do Tomcat embutido subindo na porta `8080`.
* Pressione `Ctrl + C` para encerrar o processo.

🟠 Nível 3: Passando Parâmetros e Perfis pela Linha de Comando
Cenário: Você precisa rodar a aplicação em uma porta alternativa e ativando o perfil de configuração `dev` sem alterar o código.
Sua Tarefa:
* Execute a aplicação no terminal passando o argumento de porta e o perfil ativo:
  `mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=9090" -Dspring-boot.run.profiles=dev`.
* Verifique nos logs se o Tomcat inicializou na porta 9090 e se o perfil `dev` foi exibido como ativo.

🔴 Nível 4: Descompactando e Inspecionando o Fat JAR
Cenário: Você quer comprovar a anatomia interna do arquivo executável gerado pelo Spring Boot.
Sua Tarefa:
* Execute no terminal: `mvn clean package`.
* Navegue até a pasta `target/` e copie o arquivo `.jar` gerado para uma pasta temporária renomeando a extensão para `.zip`.
* Descompacte o arquivo e navegue pela estrutura de diretórios.
* Encontre a pasta `BOOT-INF/classes` (onde está o seu código) e `BOOT-INF/lib` (onde estão todos os JARs externos).
* Abra o arquivo `META-INF/MANIFEST.MF` e localize as linhas `Main-Class` (`JarLauncher`) e `Start-Class` (`DemoApplication`).

🟣 Nível 5: Testes de Integração Automatizados com `start` e `stop`
Cenário: Você precisa subir a aplicação em segundo plano, executar testes de integração HTTP e desligá-la automaticamente ao final do build.
Sua Tarefa:
* No `pom.xml`, configure o `spring-boot-maven-plugin` com duas execuções:
  1. Goal `start` vinculado à fase `pre-integration-test`.
  2. Goal `stop` vinculado à fase `post-integration-test`.
* Crie um teste de integração simples `src/test/java/com/exemplo/HealthCheckIT.java`.
* Execute `mvn verify` e observe o Spring Boot subindo antes do teste e desligando graciosamente antes da finalização do build.

🟤 Nível 6: Gerando Informações de Build para o Actuator (`build-info`)
Cenário: A equipe de operações precisa que a rota `/actuator/info` mostre a versão do artefato e a hora exata da compilação.
Sua Tarefa:
* Adicione a dependência `spring-boot-starter-actuator` no `pom.xml`.
* No `spring-boot-maven-plugin`, adicione uma execução para o goal `build-info`.
* Execute `mvn clean compile`.
* Verifique se o arquivo `target/classes/META-INF/build-info.properties` foi criado com os metadados do projeto.

🔵 Nível 7: Injetando Informações de Git (`git-commit-id-maven-plugin`)
Cenário: Você quer auditar em produção o commit hash do Git que gerou a imagem ativa.
Sua Tarefa:
* Adicione o plugin `io.github.git-commit-id:git-commit-id-maven-plugin:7.0.0` ao `pom.xml`.
* Configure para gerar o arquivo no diretório de saída do build (`${project.build.outputDirectory}/git.properties`).
* Execute `mvn compile`.
* Inspecione o arquivo `git.properties` gerado em `target/classes/` e verifique as propriedades de commit hash, autor e branch.

🟢 Nível 8: O "Repackage Trap" em Submódulos de Biblioteca
Cenário: Você transformou seu projeto em um multimódulo com `core-lib` (sem classe `main`) e `web-api` (com classe `main`). O build falha ao tentar empacotar o `core-lib`.
Sua Tarefa:
* Observe o erro `Unable to find main class` ao executar `mvn package`.
* Abra o `core-lib/pom.xml`.
* Adicione a propriedade: `<properties><spring-boot.repackage.skip>true</spring-boot.repackage.skip></properties>`.
* Reexecute `mvn package` e comprove que o `core-lib` gerou um JAR regular de biblioteca e o `web-api` gerou o Fat JAR executável.

🟡 Nível 9: Sobrescrita de Propriedades Curadas
Cenário: O starter parent define uma versão do driver H2 ou Jackson, mas sua equipe precisa testar uma versão específica sem alterar todas as dependências.
Sua Tarefa:
* Localize o nome da propriedade da dependência no `spring-boot-dependencies` (ex: `h2.version`).
* No `<properties>` do seu `pom.xml`, declare `<h2.version>2.2.224</h2.version>`.
* Execute `mvn dependency:tree -Dincludes=com.h2database*` e comprove que a versão informada no seu POM sobrescreveu a do parent.

🟠 Nível 10: Construindo Imagem OCI/Docker sem Dockerfile (`build-image`)
Cenário: Você precisa gerar uma imagem de contêiner pronta para Kubernetes sem escrever um único arquivo `Dockerfile`.
Sua Tarefa:
* Certifique-se de que o Docker esteja em execução na sua máquina.
* No terminal do projeto, execute: `mvn spring-boot:build-image`.
* Observe o download dos Cloud Native Buildpacks (Paketo) e a criação das camadas da imagem.
* Ao finalizar, execute no terminal `docker images` e comprove que a imagem com o nome e versão do seu projeto foi criada localmente.
