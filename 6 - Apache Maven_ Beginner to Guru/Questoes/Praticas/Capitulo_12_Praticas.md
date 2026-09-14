# Questões Práticas - Capítulo 12 (Maven Repositories)

🟢 Nível 1: Inspecionando o Arquivo Global `settings.xml`
Cenário: Você quer conhecer o modelo padrão de configurações que a fundação Apache fornece com o Maven.
Sua Tarefa:
* Localize o diretório de instalação do Maven no seu sistema (`$M2_HOME` ou `${maven.home}`).
* Abra o arquivo `${maven.home}/conf/settings.xml` em um editor de texto.
* Inspecione as seções comentadas: `<proxies>`, `<servers>`, `<mirrors>` e `<profiles>`.

🟡 Nível 2: Declarando um Repositório Remoto Adicional no POM
Cenário: Você precisa utilizar uma biblioteca corporativa da Red Hat que não está publicada no Maven Central, mas reside no repositório oficial da Red Hat.
Sua Tarefa:
* Abra o `pom.xml` e crie a seção `<repositories>`.
* Adicione o repositório da Red Hat GA:
  `<repository><id>redhat-ga</id><url>https://maven.repository.redhat.com/ga/</url><snapshots><enabled>false</enabled></snapshots></repository>`.
* Adicione uma dependência da Red Hat (ex: uma extensão WildFly ou Quarkus).
* Execute `mvn compile` e observe no console o download sendo realizado a partir da URL da Red Hat.

🟠 Nível 3: Criando um Repositório Global via Profile no `settings.xml`
Cenário: Você não quer declarar a URL do repositório da Red Hat no `pom.xml` de cada projeto novo; ele deve estar disponível globalmente para a sua máquina.
Sua Tarefa:
* Abra ou crie o arquivo `${user.home}/.m2/settings.xml`.
* Crie o bloco `<profiles><profile>` com `<id>redhat-repo-profile</id>` contendo a declaração do repositório.
* Adicione o bloco `<activeProfiles><activeProfile>redhat-repo-profile</activeProfile></activeProfiles>`.
* Remova a tag `<repositories>` do seu `pom.xml` e execute `mvn help:effective-pom` para confirmar que o repositório foi incorporado globalmente.

🔴 Nível 4: Configurando um Espelho (*Mirror*) no `settings.xml`
Cenário: Você deseja que todas as buscas para o Maven Central sejam redirecionadas para um espelho mais rápido ou proxy interno.
Sua Tarefa:
* No arquivo `~/.m2/settings.xml`, abra a tag `<mirrors>`.
* Crie um `<mirror>` com `<id>meu-espelho-central</id>`, `<url>https://uk.maven.org/maven2</url>` e `<mirrorOf>central</mirrorOf>`.
* Execute um build forçando download (`mvn clean compile`) e comprove nos logs que a URL de download mudou para o espelho.

🟣 Nível 5: Simulando o Modo Offline (`-o`)
Cenário: Você está viajando de avião sem conexão com a internet e precisa compilar um projeto cuja todas as dependências já estão salvas no cache local.
Sua Tarefa:
* Desconecte seu computador da rede (ou simule).
* Execute `mvn clean package -o` (flag offline).
* Observe que o Maven não tenta fazer consultas remotas e constrói o projeto instantaneamente utilizando apenas os arquivos de `~/.m2/repository`.

🟤 Nível 6: Instalando um JAR Manualmente com `install:install-file`
Cenário: Você recebeu um arquivo legado `modulo-seguranca-1.0.jar` por e-mail que não existe em nenhum repositório público.
Sua Tarefa:
* Abra o terminal na pasta onde está o arquivo JAR.
* Execute o comando:
  `mvn install:install-file -Dfile=modulo-seguranca-1.0.jar -DgroupId=com.banco.seguranca -DartifactId=modulo-seguranca -Dversion=1.0 -Dpackaging=jar`.
* Verifique se o Maven gravou o arquivo na pasta `~/.m2/repository/com/banco/seguranca/modulo-seguranca/1.0/`.
* No `pom.xml`, declare a dependência com as coordenadas fornecidas e execute `mvn compile`.

🔵 Nível 7: Gerando a Senha Mestra do Maven
Cenário: A política de segurança da sua empresa proíbe salvar senhas de repositório em texto puro no `settings.xml`.
Sua Tarefa:
* Execute no terminal: `mvn --encrypt-master-password`.
* Digite a senha mestra desejada quando solicitado (ex: `SenhaForte123!`).
* Copie o hash cifrado retornado (ex: `{jSMO...}`).
* Crie o arquivo `~/.m2/settings-security.xml` com o conteúdo:
  `<settingsSecurity><master>{jSMO...}</master></settingsSecurity>`.

🟢 Nível 8: Criptografando a Senha do Repositório
Cenário: Com a senha mestra configurada no Nível 7, você precisa cifrar a senha de acesso ao repositório autenticado.
Sua Tarefa:
* Execute no terminal: `mvn --encrypt-password`.
* Digite a senha do seu usuário do repositório remoto quando solicitado.
* Copie o hash cifrado resultante (ex: `{COQL...}`).

🟡 Nível 9: Configurando o Servidor Autenticado no `settings.xml`
Cenário: Você precisa configurar as credenciais para o Maven acessar um repositório seguro corporativo.
Sua Tarefa:
* No arquivo `~/.m2/settings.xml`, abra a tag `<servers>`.
* Crie um bloco `<server>` com `<id>meu-repo-privado</id>`, `<username>usuario</username>` e no `<password>` cole o hash cifrado do Nível 8.
* No `pom.xml`, configure um `<repository>` com o `<id>meu-repo-privado</id>` idêntico.
* Execute `mvn compile` e comprove que o Maven descriptografa a senha em memória e autentica com sucesso.

🟠 Nível 10: Limpando Cache Local Corrompido com `dependency:purge-local-repository`
Cenário: Durante um download que caiu pela metade, um arquivo `.jar` no cache local corrompeu e o Maven insiste em falhar com erro de checksum.
Sua Tarefa:
* No terminal do projeto, execute o comando:
  `mvn dependency:purge-local-repository`.
* Observe o Maven expurgando os artefatos locais do projeto e realizando o download limpo e forçado de todas as dependências novamente a partir do repositório remoto.
