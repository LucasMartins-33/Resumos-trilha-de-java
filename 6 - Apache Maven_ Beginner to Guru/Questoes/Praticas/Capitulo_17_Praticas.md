# Questões Práticas - Capítulo 17 (Maven in the Real World)

🟢 Nível 1: Reproduzindo o Diagnóstico com `mvn dependency:tree`
Cenário: A aplicação subiu com um erro misterioso de método não encontrado e você precisa inspecionar todo o grafo de dependências.
Sua Tarefa:
* No terminal do projeto, execute o comando redirecionando a saída para um arquivo:
  `mvn dependency:tree > dependencias.txt`.
* Abra o arquivo `dependencias.txt` em um editor de texto e pesquise por termos como `swagger`, `jackson` ou `slf4j`.
* Identifique a profundidade hierárquica e as versões de cada biblioteca encontrada.

🟡 Nível 2: Filtrando Dependências Específicas na Árvore
Cenário: O arquivo completo da árvore tem milhares de linhas e você quer visualizar apenas os galhos que envolvem uma biblioteca específica.
Sua Tarefa:
* Execute no terminal o comando com o filtro `-Dincludes`:
  `mvn dependency:tree -Dincludes=org.springframework:*`.
* Observe o terminal exibindo estritamente as bibliotecas do Spring e de onde cada uma está sendo puxada.

🟠 Nível 3: Compilando e Instalando Bibliotecas Locais (`mvn clean install`)
Cenário: Você está desenvolvendo uma biblioteca compartilhada `minha-lib-core` e precisa consumi-la em um microsserviço sem publicá-la na nuvem.
Sua Tarefa:
* No diretório da biblioteca `minha-lib-core`, garanta a versão `1.0.0-SNAPSHOT` e execute:
  `mvn clean install`.
* Verifique se o JAR foi copiado para `~/.m2/repository/com/minhaempresa/minha-lib-core/1.0.0-SNAPSHOT/`.
* No microsserviço, declare a dependência com essa versão e execute `mvn compile` comprovando que o microsserviço encontrou a biblioteca local.

🔴 Nível 4: Forçando Atualização de SNAPSHOTs com a Flag `-U`
Cenário: Um colega de equipe publicou uma correção emergencial em um artefato SNAPSHOT no repositório remoto, mas seu Maven local insiste em usar a versão antiga de cache.
Sua Tarefa:
* No terminal do projeto consumidor, execute: `mvn clean package -U`.
* Observe no log do Maven a flag `--update-snapshots` forçando a verificação imediata em todos os repositórios remotos, ignorando o intervalo diário padrão.

🟣 Nível 5: Forçando Resolução de Versões com `<dependencyManagement>`
Cenário: Uma dependência transitiva profunda de terceiros está trazendo uma versão vulnerável do `jackson-databind`, e a declaração direta em `<dependencies>` não está resolvendo.
Sua Tarefa:
* No `pom.xml`, crie a seção `<dependencyManagement><dependencies>`.
* Declare o `com.fasterxml.jackson.core:jackson-databind` fixando a versão segura `2.16.1`.
* Execute `mvn dependency:tree -Dincludes=com.fasterxml.jackson.core:*` e comprove que todas as ocorrências transitivas foram forçadas para a versão 2.16.1.

🟤 Nível 6: Configurando Geração de Sources e Javadoc para o Maven Central
Cenário: Para publicar sua biblioteca de código aberto no Maven Central, é mandatório fornecer os pacotes de código-fonte e documentação.
Sua Tarefa:
* No `pom.xml`, adicione o `maven-source-plugin` com o goal `jar-no-fork` na fase `package`.
* Adicione o `maven-javadoc-plugin` com o goal `jar` na fase `package`.
* Execute `mvn clean package`.
* Inspecione a pasta `target/` e confirme a geração dos três arquivos: `app.jar`, `app-sources.jar` e `app-javadoc.jar`.

🔵 Nível 7: Configurando Assinatura Criptográfica GPG
Cenário: Todo artefato enviado ao Maven Central deve ser assinado digitalmente com uma chave PGP/GPG para comprovar autenticidade.
Sua Tarefa:
* Instale o utilitário `gpg` no seu sistema operacional e gere um par de chaves: `gpg --gen-key`.
* No `pom.xml`, adicione o `maven-gpg-plugin:3.2.0` vinculado à fase `verify` com o goal `sign`.
* Crie um perfil `<id>release-central</id>` contendo esse plugin.
* Execute `mvn verify -P release-central` e observe os arquivos de assinatura `.asc` sendo gerados na pasta `target/`.

🟢 Nível 8: Prevenindo Conflitos com `maven-enforcer-plugin` (`dependencyConvergence`)
Cenário: Você quer garantir que o Maven quebre no CI caso qualquer dependência transitiva do projeto divirja de versão.
Sua Tarefa:
* Adicione o `maven-enforcer-plugin` no `pom.xml`.
* Dentro de `<configuration><rules>`, adicione a regra vazia `<dependencyConvergence/>`.
* Execute `mvn test`.
* Se houver qualquer biblioteca com versões conflitantes na árvore, analise o relatório detalhado de convergência emitido pelo Enforcer exigindo que você alinhe as versões.

🟡 Nível 9: Banindo Bibliotecas Conflitantes (`bannedDependencies`)
Cenário: Sua organização padronizou o Logback e quer proibir estritamente que dependências antigas tragam o `log4j` ou `commons-logging` para o Classpath.
Sua Tarefa:
* No `maven-enforcer-plugin`, adicione a regra `<bannedDependencies>`.
* Na lista de `<excludes>`, adicione `log4j:log4j` e `commons-logging:commons-logging`.
* Tente adicionar propositalmente uma dependência antiga e rode `mvn compile`.
* Veja o Enforcer abortar o build e apontar a dependência banida.

🟠 Nível 10: Atualizando Projetos Downstream após a Release Oficial
Cenário: Sua biblioteca de código aberto `minha-lib-core` foi lançada oficialmente no Maven Central na versão estável `2.0.0`.
Sua Tarefa:
* Abra o `pom.xml` dos projetos consumidores (`microsservico-a` e `microsservico-b`).
* Substitua a dependência de `1.0.0-SNAPSHOT` para a versão final estável `2.0.0`.
* Execute `mvn clean verify` em ambos os projetos para garantir que a suíte completa de testes passe com a versão final distribuída.
* Commite as alterações com a mensagem `"Atualizado minha-lib-core para 2.0.0"`.
