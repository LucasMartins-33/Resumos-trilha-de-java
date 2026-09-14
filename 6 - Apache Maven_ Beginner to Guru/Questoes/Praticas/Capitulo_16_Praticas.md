# Questões Práticas - Capítulo 16 (Maven Release Plugin)

🟢 Nível 1: Configurando o `maven-release-plugin` no POM
Cenário: Você vai preparar seu projeto para suportar lançamentos formais e auditáveis de versão.
Sua Tarefa:
* Abra o `pom.xml` e certifique-se de que a versão termine em SNAPSHOT (ex: `1.0.0-SNAPSHOT`).
* Na seção de plugins do build, adicione o `org.apache.maven.plugins:maven-release-plugin:3.0.1`.
* Configure o formato da tag Git gerada dentro de `<configuration>`:
  `<tagNameFormat>v@{project.version}</tagNameFormat>`.

🟡 Nível 2: Configurando a Conexão com o Git (`<scm>`)
Cenário: O Release Plugin precisa se comunicar com o repositório Git para criar tags e commits automáticos.
Sua Tarefa:
* No `pom.xml`, crie o bloco `<scm>`.
* Configure `<connection>`: `scm:git:https://github.com/seu-usuario/meu-projeto.git`.
* Configure `<developerConnection>`: `scm:git:https://github.com/seu-usuario/meu-projeto.git` (ou formato SSH `scm:git:git@github.com:...`).
* Configure `<url>` com a URL web do repositório.

🟠 Nível 3: Simulando o Lançamento com Dry Run (`-DdryRun=true`)
Cenário: Você quer validar todo o processo de release antes de commitar qualquer alteração ou criar tags no Git.
Sua Tarefa:
* No terminal, execute: `mvn release:prepare -DdryRun=true`.
* Quando solicitado interativamente, pressione ENTER para aceitar a versão de release sugerida (`1.0.0`), o nome da tag (`v1.0.0`) e o próximo snapshot (`1.0.1-SNAPSHOT`).
* Inspecione a raiz do projeto e examine os arquivos gerados: `pom.xml.tag` e `pom.xml.next`.
* Verifique que o Git não foi alterado.

🔴 Nível 4: Limpando os Arquivos Temporários de Simulação
Cenário: Concluída a simulação do Nível 3, você precisa limpar os arquivos gerados antes de rodar o processo real.
Sua Tarefa:
* No terminal, execute o comando: `mvn release:clean`.
* Verifique com `ls` se os arquivos `pom.xml.tag`, `pom.xml.next` e `release.properties` foram completamente apagados.

🟣 Nível 5: Diagnosticando Erros Comuns: Arquivos Não Commitados
Cenário: Você tenta executar o prepare, mas esqueceu arquivos modificados na árvore de trabalho do Git.
Sua Tarefa:
* Modifique uma linha em qualquer classe Java e salve sem commitar.
* Execute `mvn release:prepare`.
* Observe a falha com a mensagem: `Cannot prepare the release because you have local modifications`.
* Faça o commit ou reverta as alterações para deixar a árvore de trabalho do Git 100% limpa.

🟤 Nível 6: Diagnosticando Erros Comuns: Dependências em SNAPSHOT
Cenário: Você tenta fazer uma release de produção, mas uma biblioteca interna ainda está apontando para uma versão SNAPSHOT.
Sua Tarefa:
* Adicione propositalmente uma dependência com versão SNAPSHOT no `pom.xml` (ex: `<version>2.0-SNAPSHOT</version>`).
* Execute `mvn release:prepare`.
* Observe o plugin interrompendo o processo e avisando que releases não podem conter dependências transitivas em SNAPSHOT.
* Remova a dependência irregular e execute `mvn release:clean`.

🔵 Nível 7: Executando o `release:prepare` Real
Cenário: Com tudo pronto e limpo, você vai executar a preparação definitiva da release no Git.
Sua Tarefa:
* Execute no terminal: `mvn release:prepare`.
* Aceite a versão de release (ex: `1.0.0`), a tag `v1.0.0` e o próximo snapshot `1.0.1-SNAPSHOT`.
* Observe o Maven rodando os testes, alterando os arquivos e gerando os commits no Git.
* Execute `git log -n 3` e `git tag` no terminal e comprove os commits automáticos e a tag criada.

🟢 Nível 8: Executando o `release:perform`
Cenário: Com a tag criada e o projeto preparado, você vai disparar o build limpo a partir da tag para efetuar o deploy.
Sua Tarefa:
* No terminal, execute: `mvn release:perform`.
* Observe o log: o Maven cria a pasta `target/checkout`, faz o checkout limpo da tag recém-criada, roda os testes novamente e dispara a fase `deploy`.
* Ao término, verifique que os arquivos de controle `release.properties` foram limpos.

🟡 Nível 9: Executando em Modo Batch sem Prompts Interativos (CI/CD)
Cenário: Você precisa rodar o Release Plugin dentro de uma esteira automatizada de CI/CD que não possui terminal interativo.
Sua Tarefa:
* Execute o comando com a flag batch mode:
  `mvn --batch-mode release:prepare release:perform`.
* Comprove que o Maven toma automaticamente as decisões padrão de versão sem parar a execução para perguntar ao usuário.

🟠 Nível 10: Prevenindo o Loop Infinito de CI com `[skip ci]`
Cenário: O pipeline de CI/CD entra em loop eterno porque cada commit automático do Release Plugin dispara um novo build de CI.
Sua Tarefa:
* No `pom.xml`, abra a tag `<configuration>` do `maven-release-plugin`.
* Adicione a configuração de prefixo de comentário:
  `<scmCommentPrefix>[maven-release-plugin] [skip ci] </scmCommentPrefix>`.
* Salve e commite.
* Entenda como a diretiva `[skip ci]` sinaliza ao CircleCI, GitHub Actions e GitLab CI para ignorar o commit automático, quebrando o loop.
