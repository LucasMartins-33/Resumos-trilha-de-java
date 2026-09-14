# Capítulo 16: Maven Release Plugin

Neste capítulo, abordamos a automação formal do ciclo de lançamento de software com o **Maven Release Plugin** (`maven-release-plugin`), a integração com sistemas de controle de versão através do **Maven SCM Plugin**, a execução de pipelines de CI/CD (CircleCI), a prevenção do temido *loop infinito de builds* e as práticas modernas de versionamento de projetos Java.

---

## 1. Conceitos Fundamentais

### 1.1 O Ciclo Formal de Release e Auditoria
No desenvolvimento profissional de software, versões de desenvolvimento (**SNAPSHOT**) são mutáveis e informais. Em contrapartida, uma **Release** é definitiva e deve obedecer a rígidos critérios de conformidade e governança:
1. **Rastreabilidade Bidirecional**: Deve ser possível olhar para um arquivo `.jar` em produção e apontar exatamente qual commit e tag no Git gerou aquele binário.
2. **Imutabilidade**: Dependências não podem oscilar; uma release não pode conter dependências transientes em SNAPSHOT.
3. **Automação**: Processos manuais de alteração de versão em arquivos POM geram erros humanos frequentes.

---

### 1.2 Os Dois Passos Fundamentais do Release Plugin

```text
1. release:prepare
   ├── Verifica modificações locais não commitadas (aborta se houver)
   ├── Verifica dependências SNAPSHOT de terceiros (aborta se houver)
   ├── Altera o POM: Remove "-SNAPSHOT" (ex: 1.0.0-SNAPSHOT ➔ 1.0.0)
   ├── Executa a suíte de testes
   ├── Cria o Commit e a Tag no Git (ex: v1.0.0)
   ├── Altera o POM: Incrementa para a próxima versão (ex: 1.0.1-SNAPSHOT)
   ├── Faz o Commit e Push das alterações de versão para o repositório remoto
   └── Gera arquivos temporários: release.properties e pom.xml.releaseBackup

2. release:perform
   ├── Clona o projeto limpo a partir da Tag Git criada em target/checkout
   ├── Executa os testes novamente para verificação final
   ├── Executa os goals de deploy (sobe o JAR para o Packagecloud, Nexus ou Central)
   └── Remove os arquivos transitórios de controle (release.properties)
```

---

### 1.3 Objetivos de Suporte do Release Plugin

* **`release:rollback`**:
  * Reverte as alterações de versão no `pom.xml` e apaga os arquivos de backup caso o `release:prepare` tenha falhado ou sido cancelado.
  * **Atenção**: Não funciona se o comando `release:clean` já tiver sido executado.
  * **Limitação Histórica**: O plugin nem sempre remove a tag remota criada no Git; a tag frequentemente precisa ser deletada manualmente (`git push --delete origin <tag>`).
* **`release:clean`**:
  * Remove os arquivos transitórios (`release.properties`, backups de POM) deixados por execuções interrompidas.
* **`release:prepare -DdryRun=true`**:
  * Modo de simulação (*Dry Run*). Não altera o Git nem faz commits; gera os arquivos de pré-visualização `pom.xml.tag` (como ficará na release) e `pom.xml.next` (como ficará no próximo snapshot), permitindo validação prévia em projetos complexos.

---

### 1.4 Configuração do Maven SCM (Source Control Management)
O Release Plugin utiliza internamente o **Maven SCM Plugin** para interagir com o Git:
* `<connection>`: URL de leitura pública (usada para consulta).
* `<developerConnection>`: URL de escrita autenticada (`scm:git:https://github.com/...` ou `scm:git:git@github.com:...`), utilizada pelo plugin para commitar tags e versões.
* `<project.scm.id>`: Propriedade que mapeia o ID do servidor configurado em `settings.xml` para injetar as credenciais do Git.

---

### 1.5 O Problema do "Loop Infinito" em Servidores de CI/CD
Ao automatizar o `release:prepare` dentro de uma esteira de CI/CD (CircleCI, GitHub Actions, Jenkins):
1. O desenvolvedor dá push em `master`.
2. O CircleCI detecta a alteração e inicia o build.
3. O Maven Release Plugin roda e faz **dois novos commits** de incremento de versão no `master`.
4. O CircleCI detecta os novos commits e dispara um **segundo build**.
5. O segundo build executa o plugin, faz novos commits e inicia um **loop infinito de releases** descontroladas!

#### A Solução: Configurar `<scmCommentPrefix>`
Para impedir o loop, instrui-se o plugin a adicionar a diretiva de escape do CI na mensagem dos commits gerados automaticamente:
```xml
<configuration>
    <scmCommentPrefix>[maven-release-plugin] [skip ci]</scmCommentPrefix>
</configuration>
```
O servidor de CI lê a marcação `[skip ci]` e ignora o commit, quebrando o ciclo vicioso.

---

## 2. Sintaxe, Comandos & Configurações

### 2.1 Configuração Completa no `pom.xml`

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>guru.springframework</groupId>
    <artifactId>mb2g-release-plugin</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <!-- Aponta para o bloco <server> no settings.xml para login no Git -->
        <project.scm.id>github-server</project.scm.id>
    </properties>

    <!-- Configuração do Controle de Versão (SCM) -->
    <scm>
        <connection>scm:git:https://github.com/usuario/mb2g-release-plugin.git</connection>
        <developerConnection>scm:git:https://github.com/usuario/mb2g-release-plugin.git</developerConnection>
        <url>https://github.com/usuario/mb2g-release-plugin</url>
        <tag>HEAD</tag>
    </scm>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-release-plugin</artifactId>
                <version>3.0.1</version>
                <configuration>
                    <!-- Evita loop infinito no CI/CD -->
                    <scmCommentPrefix>[maven-release-plugin] [skip ci] </scmCommentPrefix>
                    <!-- Em projetos multimódulo, unifica a versão de todos os submódulos -->
                    <autoVersionSubmodules>true</autoVersionSubmodules>
                    <tagNameFormat>v@{project.version}</tagNameFormat>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

---

### 2.2 Configuração de Credenciais Seguras no `settings.xml`
Injeção dinâmica de segredos através de variáveis de ambiente do runner de CI:

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" ...>
    <servers>
        <!-- Credenciais do GitHub para permitir push de commits e tags pelo SCM Plugin -->
        <server>
            <id>github-server</id>
            <username>${env.GH_USERNAME}</username>
            <password>${env.GH_TOKEN}</password> <!-- Token de Acesso Pessoal (PAT) -->
        </server>
        
        <!-- Credenciais do repositório remoto para publicação do JAR -->
        <server>
            <id>packagecloud.release</id>
            <password>${env.PACKAGECLOUD_TOKEN}</password>
        </server>
    </servers>
</settings>
```

---

### 2.3 Exemplo de Pipeline CircleCI (`.circleci/config.yml`)
```yaml
version: 2.1
jobs:
  build-and-release:
    docker:
      - image: cimg/openjdk:17.0
    steps:
      - checkout
      - run:
          name: Configurar Identidade Git no Runner
          command: |
            git config --global user.email "ci-bot@empresa.com"
            git config --global user.name "CI Automation Bot"
      - run:
          name: Executar Release Maven em Batch Mode
          command: |
            mvn --batch-mode release:prepare release:perform \
              -s .circleci/settings.xml
workflows:
  release-workflow:
    jobs:
      - build-and-release:
          filters:
            branches:
              only: master
```

---

## 3. Tabela de Comandos do Capítulo

| Comando | Descrição |
| :--- | :--- |
| `mvn release:prepare` | Valida o projeto, roda testes, altera versões no POM e gera a Tag no Git. |
| `mvn release:perform` | Clona a partir da Tag gerada e realiza o `deploy` dos artefatos. |
| `mvn --batch-mode release:prepare release:perform` | Executa o fluxo completo sem interatividade, aceitando as versões sugeridas (essencial para CI/CD). |
| `mvn release:prepare -DdryRun=true` | Executa simulação prévia sem efetuar commits ou alterar o Git. |
| `mvn release:clean` | Limpa os arquivos temporários `release.properties` e backups de POM. |
| `mvn release:rollback` | Reverte o POM para o estado pré-release (caso `release:clean` não tenha sido chamado). |
| `mvn release:update-versions -DautoVersionSubmodules=true` | Incrementa/atualiza versões em projetos multimódulos sem disparar release completa. |

---

## 4. Apêndice — Atualizações & Boas Práticas Modernas

### 4.1 O Declínio do `maven-release-plugin` em Ambientes Modernos
Embora o Maven Release Plugin tenha sido o padrão da indústria por mais de uma década, no desenvolvimento moderno orientado a nuvem e **GitOps** ele passou a enfrentar graves restrições:
1. **Regras de Proteção de Branches (*Branch Protection Rules*)**:
   * Em quase todas as empresas modernas, a branch `main`/`master` é bloqueada para escrita direta. Ninguém (nem bots de CI) pode dar push direto sem abrir um Pull Request aprovado por revisores. Como o Release Plugin exige fazer `git push` direto na branch principal, ele falha a menos que permissões administrativas perigosas de bypass sejam concedidas.
2. **Modificação Suja do Git**:
   * Fazer commits de release e de próximo snapshot polui desnecessariamente o histórico do Git com mensagens puramente cosméticas (`[maven-release-plugin] prepare for next development iteration`).

---

### 4.2 A Abordagem Moderna: Versões Amigáveis ao CI (`CI-Friendly Versions`)
Desde o **Maven 3.5.0**, a fundação Apache introduziu suporte nativo às propriedades `${revision}`, `${sha1}` e `${changelist}`:
```xml
<groupId>com.empresa</groupId>
<artifactId>meu-servico</artifactId>
<version>${revision}</version>
```
* O `pom.xml` nunca é alterado fisicamente no Git! O arquivo mantém sempre `<version>${revision}</version>`.
* Em ambiente de desenvolvimento local: `${revision}` pode ter valor default `1.0.0-SNAPSHOT`.
* No pipeline de CI/CD: A versão é passada como parâmetro limpo:
  ```bash
  mvn clean deploy -Drevision=1.4.2
  ```
* O plugin **`flatten-maven-plugin`** é acoplado para gerar o POM publicado no repositório com a versão resolvida estática (`1.4.2`), sem poluir o repositório Git.

---

### 4.3 Ferramentas Modernas de Automação de Release
A indústria moderna substituiu a mecânica do Release Plugin por:
1. **Releases Baseadas em Tags no Git (*Tag-Driven Releases*)**:
   * O desenvolvedor gera uma tag no Git (ex: `git tag v2.1.0 && git push origin v2.1.0`).
   * O CI/CD detecta o evento da tag, extrai o número da versão e executa `mvn clean deploy -Drevision=2.1.0` diretamente.
2. **JReleaser**:
   * O utilitário moderno mais popular do ecossistema Java. Gerencia criação de changelogs a partir de PRs, publicação no Maven Central, GitHub Releases, Homebrew, SDKMAN! e Docker Hub de forma declarativa e desacoplada do código do POM.
3. **Semantic Release & Conventional Commits**:
   * Analisa mensagens de commit padronizadas (`fix:`, `feat:`, `feat!:`) para calcular automaticamente a versão semântica (Patch, Minor ou Major) e publicar a release sem intervenção humana.
