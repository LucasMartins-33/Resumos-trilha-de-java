# Questões Práticas - Capítulo 15 (Maven Build Profiles)

🟢 Nível 1: Declarando o Primeiro Build Profile no POM
Cenário: Você quer criar um perfil para compilar uma versão de testes com empacotamento especial sem poluir a configuração base.
Sua Tarefa:
* Abra o `pom.xml` e crie a seção `<profiles>`.
* Crie um `<profile>` com o `<id>dev</id>`.
* Dentro dele, declare a tag `<properties><app.ambiente>Desenvolvimento</app.ambiente></properties>`.
* Execute `mvn help:active-profiles` no terminal e observe que o perfil `dev` **não** está ativo por padrão.

🟡 Nível 2: Ativando Perfis Manualmente via Linha de Comando (`-P`)
Cenário: Você quer testar a ativação do perfil criado no Nível 1 durante o ciclo de build.
Sua Tarefa:
* No terminal, execute: `mvn compile -P dev`.
* Execute `mvn help:active-profiles -P dev`.
* Comprove no console que o Maven lista o perfil `dev` como ativo para a execução atual.

🟠 Nível 3: Ativação Automática Padrão (`<activeByDefault>`)
Cenário: Você quer que o perfil `dev` esteja sempre ativo, a menos que outro perfil seja expressamente solicitado.
Sua Tarefa:
* Dentro do `<profile>` com `<id>dev</id>`, adicione o bloco:
  `<activation><activeByDefault>true</activeByDefault></activation>`.
* Execute `mvn help:active-profiles` (sem passar nenhum argumento `-P`).
* Verifique que o perfil `dev` agora aparece automaticamente na lista de perfis ativos.

🔴 Nível 4: Desativando um Perfil Ativo por Padrão
Cenário: Você configurou o perfil `dev` como ativo por padrão, mas precisa rodar um build limpo sem ele.
Sua Tarefa:
* No terminal, execute o comando de desativação:
  `mvn compile -P \!dev` (se estiver no Linux/macOS Bash) ou `mvn compile -P -dev`.
* Observe a saída do `help:active-profiles` comprovando que o perfil `dev` foi desativado.

🟣 Nível 5: Alternando Destinos de Deploy com Perfis
Cenário: Você precisa enviar o projeto ora para o Packagecloud, ora para o Nexus corporativo dependendo de onde o build é disparado.
Sua Tarefa:
* Crie dois perfis no `pom.xml`: `deploy-nexus` e `deploy-packagecloud`.
* Mova a configuração de `<distributionManagement>` do Nexus para dentro do perfil `deploy-nexus`.
* Coloque a configuração de `<distributionManagement>` do Packagecloud dentro do perfil `deploy-packagecloud`.
* Remova o bloco `<distributionManagement>` da raiz do POM.
* Teste a chamada: `mvn clean deploy -P deploy-nexus -DskipTests` e comprove o envio ao Nexus.

🟤 Nível 6: Injetando Propriedades de Ambiente em Testes Automatizados
Cenário: Seus testes de integração precisam disparar chamadas contra `localhost` no desktop dos desenvolvedores e contra `qa.api.empresa.com` no ambiente de testes.
Sua Tarefa:
* No `<properties>` raiz do POM, defina `<TEST_HOST>localhost:8080</TEST_HOST>`.
* Crie o perfil `<id>qa</id>` sobrescrevendo `<TEST_HOST>qa.api.empresa.com</TEST_HOST>`.
* No `maven-surefire-plugin`, configure:
  `<environmentVariables><TEST_HOST>${TEST_HOST}</TEST_HOST></environmentVariables>`.
* Crie um teste que faça `System.out.println("Host: " + System.getenv("TEST_HOST"))`.
* Execute `mvn test` (deve imprimir localhost) e depois `mvn test -P qa` (deve imprimir qa.api...).

🔵 Nível 7: Ativação Automática por Versão do JDK
Cenário: Você quer que certas otimizações de compilação ou flags de preview só sejam ativadas quando o build rodar em Java 21 ou superior.
Sua Tarefa:
* Crie um perfil com `<id>java21-flags</id>`.
* Na seção `<activation>`, adicione `<jdk>21</jdk>` (ou `[21,)`).
* Dentro do perfil, adicione o argumento de compilador `--enable-preview`.
* Execute o build usando o JDK 17 e comprove que o perfil permanece inativo. Alterne para o JDK 21 e veja o perfil sendo ativado automaticamente.

🟢 Nível 8: Ativação Condicional por Sistema Operacional
Cenário: Um script utilitário de terminal no seu projeto gera um script `.bat` no Windows e um script `.sh` no Linux/macOS.
Sua Tarefa:
* Crie dois perfis: `windows-config` (com `<activation><os><family>windows</family></os></activation>`) e `unix-config` (com `<activation><os><family>unix</family></os></activation>`).
* Em cada perfil, configure uma propriedade `<script.ext>` com `.bat` ou `.sh`.
* Execute `mvn help:active-profiles` no seu sistema operacional atual e comprove que o Maven ativou apenas o perfil correto da sua plataforma.

🟡 Nível 9: Ativação Condicional por Arquivo em Disco
Cenário: Se o desenvolvedor tiver um arquivo secreto `local-secrets.properties` na raiz da pasta, o Maven deve ler esse arquivo para injetar credenciais de banco.
Sua Tarefa:
* Crie um perfil com `<id>secrets-ativo</id>`.
* Na tag `<activation>`, use `<file><exists>local-secrets.properties</exists></file>`.
* Crie o arquivo de texto `local-secrets.properties` na pasta do projeto.
* Execute `mvn help:active-profiles` e veja o perfil ser ativado.
* Apague ou renomeie o arquivo e comprove que o perfil deixa de ser ativado.

🟠 Nível 10: O Perfil de CI Estrito para Pipelines
Cenário: Em máquinas de desenvolvimento, você quer compilações rápidas, mas no GitHub Actions/GitLab CI você quer exigir JaCoCo, SpotBugs e falhar o build em qualquer aviso.
Sua Tarefa:
* Crie o perfil `<id>ci</id>`.
* Dentro do perfil, configure o `jacoco-maven-plugin` com a meta `check` exigindo 80% de cobertura e o `spotbugs-maven-plugin` com a meta `check`.
* Execute `mvn clean verify` localmente (deve rodar rápido sem SpotBugs).
* Execute `mvn clean verify -P ci` simulando a esteira de CI e comprove a execução rigorosa de todas as análises estáticas e de cobertura.
