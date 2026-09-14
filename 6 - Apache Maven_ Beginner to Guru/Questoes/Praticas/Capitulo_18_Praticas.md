# Questões Práticas - Capítulo 18 (Enterprise Dependency Management)

🟢 Nível 1: Inicializando o Repositório do Parent BOM Corporativo
Cenário: Você é o arquiteto de software responsável por padronizar as dependências de 20 microsserviços da empresa criando o projeto `empresa-parent-bom`.
Sua Tarefa:
* Crie uma nova pasta de projeto e inicialize um arquivo `pom.xml`.
* Defina as coordenadas: `groupId: com.minhaempresa.bom`, `artifactId: enterprise-bom`, `version: 1.0.0-SNAPSHOT`.
* Configure obrigatoriamente a tag `<packaging>pom</packaging>`.
* Delete qualquer diretório `src/` caso tenha sido gerado.

🟡 Nível 2: Herdando do `spring-boot-starter-parent` no BOM
Cenário: Sua empresa adota o Spring Boot como padrão. O seu BOM corporativo deve herdar do parent oficial do Spring para aproveitar as curadorias base.
Sua Tarefa:
* No `enterprise-bom/pom.xml`, adicione a seção `<parent>`.
* Configure as coordenadas do `org.springframework.boot:spring-boot-starter-parent:3.2.4`.
* Defina propriedades corporativas globais no `<properties>`:
  `<java.version>17</java.version>` e `<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>`.

🟠 Nível 3: Centralizando Versões de Terceiros em `<dependencyManagement>`
Cenário: Todos os microsserviços utilizam MapStruct e drivers de banco, e você quer definir essas versões em um único lugar.
Sua Tarefa:
* No `enterprise-bom`, crie a tag `<properties><mapstruct.version>1.5.5.Final</mapstruct.version></properties>`.
* No bloco `<dependencyManagement><dependencies>`, declare `org.mapstruct:mapstruct` com a versão parametrizada.
* Declare também o driver `com.mysql:mysql-connector-j:8.3.0`.

🔴 Nível 4: Injetando Dependências Universais Obrigatórias
Cenário: A equipe de segurança e observabilidade exige que **todos** os microsserviços tenham o Actuator e o Lombok incluídos obrigatoriamente.
Sua Tarefa:
* No `enterprise-bom`, crie o bloco de dependências diretas `<dependencies>`.
* Adicione `spring-boot-starter-actuator`.
* Adicione `org.projectlombok:lombok` (com `<optional>true</optional>`).
* Adicione `spring-boot-starter-test` com escopo `test`.

🟣 Nível 5: Configurando Compilação Unificada de Anotações (Lombok + MapStruct)
Cenário: Você quer poupar os desenvolvedores de configurarem o compilador repetidamente em cada microsserviço.
Sua Tarefa:
* Na seção `<build><plugins>` do `enterprise-bom`, configure o `maven-compiler-plugin`.
* Dentro de `<annotationProcessorPaths>`, declare o processador do MapStruct, o processador do Lombok e o `lombok-mapstruct-binding`.
* Adicione o argumento do compilador: `<compilerArg>-Amapstruct.defaultComponentModel=spring</compilerArg>`.

🟤 Nível 6: Impondo Regras Corporativas com o Enforcer Plugin
Cenário: A governança da empresa exige que nenhum desenvolvedor compile em Java 8/11 e que todos usem o Maven 3.6.3 ou superior.
Sua Tarefa:
* Adicione o `maven-enforcer-plugin` no `enterprise-bom`.
* Configure as regras:
  * `<requireMavenVersion><version>[3.6.3,)</version></requireMavenVersion>`.
  * `<requireJavaVersion><version>[17,)</version></requireJavaVersion>`.
  * `<requireReleaseDeps><onlyWhenRelease>true</onlyWhenRelease></requireReleaseDeps>`.

🔵 Nível 7: Instalando o BOM no Cache Local (`mvn install`)
Cenário: Para que os outros microsserviços no seu computador consigam enxergar o novo BOM, ele precisa ser registrado localmente.
Sua Tarefa:
* Abra o terminal na pasta do `enterprise-bom`.
* Execute: `mvn clean install`.
* Inspecione a pasta `~/.m2/repository/com/minhaempresa/bom/enterprise-bom/1.0.0-SNAPSHOT/` e comprove a presença do arquivo `.pom` instalado.

🟢 Nível 8: Refatorando um Microsserviço para Usar o Parent BOM
Cenário: Você vai pegar o microsserviço `pagamento-service` e eliminar centenas de linhas de código repetitivo de configuração.
Sua Tarefa:
* No `pagamento-service/pom.xml`, altere o bloco `<parent>` para apontar para o `com.minhaempresa.bom:enterprise-bom:1.0.0-SNAPSHOT`.
* Delete do POM do serviço: declarações de versão do Java, configurações do compilador, dependências do Lombok, Actuator e suítes de teste básicas (pois tudo é herdado!).
* Execute `mvn clean verify` no serviço e confirme que tudo compila e os testes passam com sucesso.

🟡 Nível 9: Configurando um Workspace Multiprojetos no IntelliJ IDEA
Cenário: Você precisa desenvolver e depurar alterações simultâneas em 3 microsserviços independentes (`cliente-service`, `pedido-service` e `pagamento-service`) na mesma janela da IDE.
Sua Tarefa:
* No IntelliJ IDEA, vá em *File* ➔ *New* ➔ *Project...* e escolha **Empty Project**.
* Nomeie como `empresa-microservices-workspace`.
* Vá em *File* ➔ *New* ➔ *Module from Existing Sources...* e selecione o `pom.xml` de cada um dos três microsserviços.
* Abra a aba do Maven na IDE e comprove que os três projetos aparecem independentes com seus respectivos ciclos de vida.

🟠 Nível 10: Composição Moderna de BOMs sem Herança Única (`<scope>import</scope>`)
Cenário: O microsserviço precisa das regras do seu BOM corporativo, mas também precisa herdar de um framework específico de terceiros que impede o uso de `<parent>`.
Sua Tarefa:
* No `pom.xml` do microsserviço, remova o `<parent>` do `enterprise-bom`.
* No bloco `<dependencyManagement><dependencies>`, importe o BOM corporativo:
  `<dependency><groupId>com.minhaempresa.bom</groupId><artifactId>enterprise-bom</artifactId><version>1.0.0</version><type>pom</type><scope>import</scope></dependency>`.
* Execute `mvn clean compile` e valide que o microsserviço herdou as versões homologadas via composição, sem restrição de herança única de pais.
