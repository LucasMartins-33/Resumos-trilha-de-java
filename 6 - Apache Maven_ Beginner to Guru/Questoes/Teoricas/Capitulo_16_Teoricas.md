# Questões Teóricas - Capítulo 16 (Maven Release Plugin)

**1. O Propósito do Maven Release Plugin:** Qual é o objetivo primário do `maven-release-plugin` e quais tarefas manuais repetitivas e propensas a erro ele automatiza?
> [!faq]- 👀 Ver Resposta
> O objetivo é padronizar e automatizar o ciclo formal de lançamento de software em conformidade com as regras de auditoria corporativa. Ele automatiza: a checagem de código não commitado, a validação de que não existem dependências SNAPSHOT residuais, a remoção do sufixo `-SNAPSHOT` do `pom.xml`, a execução completa dos testes, a criação da Tag no sistema de controle de versão (Git), o incremento para o próximo número de versão SNAPSHOT, o commit das alterações no Git e o deploy final do artefato no repositório remoto.

**2. O Objetivo `release:prepare`:** Descreva em detalhes as verificações e passos que o Maven executa durante o goal `mvn release:prepare`.
> [!faq]- 👀 Ver Resposta
> O `release:prepare` executa sequencialmente:
> 1. Verifica se há arquivos modificados localmente e não commitados no Git (aborta se houver).
> 2. Verifica se o projeto depende de artefatos SNAPSHOT de terceiros (aborta se houver).
> 3. Altera a versão no `pom.xml` para a versão final de release (ex: `1.0.0`).
> 4. Executa a suíte de testes unitários para certificar que o código está íntegro.
> 5. Cria o commit de release e a Tag correspondente no Git (ex: `v1.0.0`).
> 6. Incrementa a versão no `pom.xml` para o próximo snapshot (ex: `1.0.1-SNAPSHOT`).
> 7. Commita a nova versão de desenvolvimento no Git e envia (*push*) as tags e commits para o repositório remoto.
> 8. Gera arquivos temporários de controle: `release.properties` e backups do POM.

**3. O Objetivo `release:perform`:** O que acontece nos bastidores quando o desenvolvedor executa `mvn release:perform` após um prepare bem-sucedido?
> [!faq]- 👀 Ver Resposta
> O `release:perform` lê as informações gravadas em `release.properties`, clona uma cópia limpa do projeto diretamente a partir da **Tag Git** recém-criada para a pasta temporária `target/checkout`, executa os testes novamente para validação estrita, executa os objetivos de empacotamento e deploy (`mvn deploy site-deploy`), publicando o JAR oficial de release no repositório remoto (Nexus, Packagecloud ou Maven Central), e por fim limpa os arquivos temporários.

**4. A Flag `-DdryRun=true`:** Para que serve a opção `mvn release:prepare -DdryRun=true` e quais arquivos especiais de pré-visualização ela produz em disco?
> [!faq]- 👀 Ver Resposta
> Trata-se de um modo de simulação (*Dry Run*). Ela executa todas as análises e cálculos do plugin sem alterar o Git e sem fazer commits ou tags. Ela gera no diretório do projeto os arquivos `pom.xml.tag` (mostrando como o POM ficará na versão oficial de release) e `pom.xml.next` (mostrando como o POM ficará no próximo ciclo de desenvolvimento SNAPSHOT), permitindo que desenvolvedores revisem o resultado antes de executar a release definitiva.

**5. O Objetivo `release:rollback` e suas Limitações:** Qual é a função do `mvn release:rollback` quando um prepare falha e qual é sua limitação histórica conhecida em relação a Tags no Git?
> [!faq]- 👀 Ver Resposta
> O `release:rollback` utiliza os arquivos de backup gerados para restaurar o `pom.xml` ao seu estado original antes da tentativa de release e reverter os commits locais de versão. Suas duas limitações críticas são:
> 1. Ele não funciona se o comando `release:clean` já tiver sido executado (pois este apaga os arquivos de backup).
> 2. O plugin frequentemente **não remove a Tag remota criada no Git**, exigindo que o desenvolvedor execute manualmente a exclusão da tag no Git (`git push --delete origin <tag>`).

**6. Configuração do Bloco `<scm>` no `pom.xml`:** Qual é a diferença entre a tag `<connection>` e a tag `<developerConnection>` dentro da seção `<scm>` de um projeto Maven?
> [!faq]- 👀 Ver Resposta
> * `<connection>`: Especifica a URL de acesso somente-leitura ao repositório de código (geralmente pública ou anônima via HTTPS).
> * `<developerConnection>`: Especifica a URL de acesso autenticado com permissão de escrita/push (geralmente via SSH `scm:git:git@github.com:...` ou HTTPS autenticado). O `maven-release-plugin` utiliza obrigatoriamente a URL do `developerConnection` para empurrar tags e novos commits de volta ao repositório.

**7. O "Loop Infinito" de Builds em Servidores de CI/CD:** O que causa o fenômeno do "loop infinito" quando o Maven Release Plugin é executado dentro de uma pipeline de CI/CD como o CircleCI ou Jenkins?
> [!faq]- 👀 Ver Resposta
> O pipeline detecta um push na branch `main` e inicia a esteira de CI. Durante a execução, o `release:prepare` gera commits de alteração de versão e faz o `git push` de volta para a mesma branch `main`. O servidor de CI detecta esses novos commits e assume que houve uma alteração de código, disparando um novo build imediatamente. O novo build executa o plugin novamente, gerando mais commits e reiniciando o ciclo de forma descontrolada.

**8. A Solução do Loop com `<scmCommentPrefix>`:** Como a inclusão de marcas como `[skip ci]` na configuração do plugin soluciona o problema do loop infinito em esteiras de integração contínua?
> [!faq]- 👀 Ver Resposta
> Configura-se o plugin com:
> ```xml
> <scmCommentPrefix>[maven-release-plugin] [skip ci] </scmCommentPrefix>
> ```
> O plugin prefixará todas as mensagens de commit automáticas geradas por ele com a diretiva `[skip ci]`. Praticamente todos os servidores modernos de CI/CD (CircleCI, GitHub Actions, GitLab CI) inspecionam a mensagem de cada novo commit; ao identificarem a instrução `[skip ci]`, eles ignoram o evento e não iniciam novas esteiras, quebrando o ciclo com sucesso.

**9. A Incompatibilidade Moderna com Regras de Proteção de Branches:** Por que a utilização clássica do `maven-release-plugin` tornou-se inviável em organizações modernas que utilizam *Branch Protection Rules* no GitHub/GitLab?
> [!faq]- 👀 Ver Resposta
> Em empresas modernas, a branch principal (`main`/`master`) é rigorosamente protegida contra escrita direta: ninguém (nem programadores nem esteiras comuns de CI) pode fazer `git push` diretamente nela sem passar por um Pull Request com aprovações humanas obrigatórias de code review. Como o `release:prepare` tenta dar push direto na branch principal para commitar as alterações de POM, o Git rejeita a operação com erro de permissão.

**10. A Abordagem Moderna: Versões Amigáveis ao CI (`CI-Friendly Versions`):** Como o recurso de CI-Friendly Versions (introduzido no Maven 3.5+) e a propriedade `${revision}` substituíram a necessidade de alterar arquivos POM no Git?
> [!faq]- 👀 Ver Resposta
> Em vez de commitar números fixos no POM, declara-se a versão como `<version>${revision}</version>`. O arquivo `pom.xml` nunca mais é alterado fisicamente no Git. No desenvolvimento local, `${revision}` assume o valor padrão `1.0.0-SNAPSHOT`. Na esteira de CI/CD, a release é acionada por uma Git Tag (ex: `v2.1.0`), e o pipeline simplesmente executa:
> ```bash
> mvn clean deploy -Drevision=2.1.0
> ```
> O `flatten-maven-plugin` gera em memória o POM final publicado com a versão real, eliminando a poluição do histórico Git e a necessidade do complexo ciclo de prepare/perform.
