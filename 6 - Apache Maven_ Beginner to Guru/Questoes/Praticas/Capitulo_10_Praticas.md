# Questões Práticas - Capítulo 10 (Multi-Module Projects)

🟢 Nível 1: Criando a Raiz do Projeto Multimódulo (Aggregator POM)
Cenário: Você foi encarregado de iniciar a arquitetura de um novo sistema corporativo modularizado chamado `sistema-vendas`.
Sua Tarefa:
* Crie uma pasta raiz chamada `sistema-vendas` e crie um arquivo `pom.xml` dentro dela.
* Defina as coordenadas: `groupId: com.empresa.vendas`, `artifactId: sistema-vendas-parent`, `version: 1.0-SNAPSHOT`.
* Configure obrigatoriamente a tag `<packaging>pom</packaging>`.
* Não crie diretórios `src/` na raiz.

🟡 Nível 2: Criando o Primeiro Submódulo (`vendas-model`)
Cenário: O modelo de domínio e as entidades de banco precisam residir em um módulo isolado e reutilizável.
Sua Tarefa:
* Crie uma subpasta `vendas-model` dentro da raiz.
* Crie o arquivo `vendas-model/pom.xml`.
* No `pom.xml` do submódulo, configure o bloco `<parent>` apontando para o `sistema-vendas-parent`.
* No POM raiz da pasta principal, adicione o bloco `<modules><module>vendas-model</module></modules>`.
* Crie a pasta `vendas-model/src/main/java` e adicione a classe `Produto.java`.

🟠 Nível 3: Executando o Build pelo Reactor do Maven
Cenário: Você quer validar que o Maven reconhece a estrutura agregadora e compila o submódulo a partir da raiz.
Sua Tarefa:
* Posicione seu terminal na pasta raiz `sistema-vendas`.
* Execute `mvn clean compile`.
* Analise a tabela **Reactor Build Order** exibida no console e verifique a ordem de compilação: primeiro o parent raiz e depois o `vendas-model`.

🔴 Nível 4: Criando o Segundo Submódulo Dependente (`vendas-service`)
Cenário: A camada de serviços de negócio precisa consumir as classes do modelo de dados.
Sua Tarefa:
* Crie a subpasta `vendas-service` com seu respectivo `pom.xml` herdando do parent raiz.
* Registre o novo módulo na tag `<modules>` do POM raiz.
* Dentro de `vendas-service/pom.xml`, adicione a dependência para o módulo irmão:
  `<dependency><groupId>com.empresa.vendas</groupId><artifactId>vendas-model</artifactId><version>${project.version}</version></dependency>`.
* Crie a classe `EstoqueService.java` dentro do serviço instanciando a classe `Produto`.

🟣 Nível 5: Validando a Resolução Automática do Reactor
Cenário: Você quer observar o cálculo topológico de compilação do Maven em ação.
Sua Tarefa:
* Inverta propositalmente a ordem dos módulos no POM raiz para:
  `<modules><module>vendas-service</module><module>vendas-model</module></modules>`.
* Execute `mvn clean compile` a partir da raiz.
* Observe o console: mesmo listado depois no XML, o Maven compila o `vendas-model` primeiro porque detectou que o `vendas-service` depende dele.

🟤 Nível 6: Padronizando Versões com `<dependencyManagement>` no Pai
Cenário: Você não quer que cada submódulo declare versões manuais de bibliotecas externas (como Spring ou Jackson).
Sua Tarefa:
* No POM pai raiz, abra o bloco `<dependencyManagement><dependencies>`.
* Declare `org.apache.commons:commons-lang3` com a versão `3.12.0`.
* Nos submódulos `vendas-model` e `vendas-service`, declare a dependência do `commons-lang3` **sem a tag `<version>`**.
* Execute `mvn compile` a partir da raiz e comprove a herança de versão.

🔵 Nível 7: Compilação Seletiva com a Flag `-pl` (Project List)
Cenário: Você alterou apenas uma linha no `vendas-service` e não quer esperar o Maven recompilar o projeto inteiro.
Sua Tarefa:
* Estando na pasta raiz do projeto, execute o comando:
  `mvn compile -pl vendas-service`.
* Observe a saída do console: o Maven executará a compilação exclusivamente no módulo `vendas-service`, ignorando os demais.

🟢 Nível 8: Compilando com Dependências Upstream (`-am` / Also Make)
Cenário: Você alterou tanto o `vendas-model` quanto o `vendas-service`. Se compilar apenas o service com `-pl`, as alterações do model não serão recompiladas.
Sua Tarefa:
* Execute a partir da raiz: `mvn compile -pl vendas-service -am`.
* Observe o log: o Reactor detecta as dependências e compila primeiro o `vendas-model` (porque o service precisa dele) e depois o `vendas-service`.

🟡 Nível 9: Criando o Módulo Web Final Executável (`vendas-web`)
Cenário: Você vai criar o ponto de entrada da aplicação que expõe a API HTTP e empacota o JAR executável.
Sua Tarefa:
* Crie o módulo `vendas-web` registrado no POM raiz.
* Adicione a dependência do `vendas-service` no `vendas-web/pom.xml`.
* Configure o `vendas-web` com a classe `MainApplication.java`.
* Execute `mvn clean package` a partir da raiz.
* Inspecione a pasta `vendas-web/target/` e verifique que o JAR final foi gerado contendo o acesso a todos os módulos anteriores.

🟠 Nível 10: Retomando um Build que Falhou (`--resume-from` / `-rf`)
Cenário: Durante um build longo de 10 módulos, o módulo `vendas-web` falhou por um erro de digitação pontual após 5 minutos de compilação dos módulos anteriores.
Sua Tarefa:
* Corrija o erro no arquivo do `vendas-web`.
* Em vez de executar `mvn clean package` e recompilar tudo do zero, execute:
  `mvn package -rf :vendas-web` (ou `-rf vendas-web`).
* Observe que o Maven ignora os módulos anteriores já compilados e retoma o build instantaneamente a partir do `vendas-web`.
