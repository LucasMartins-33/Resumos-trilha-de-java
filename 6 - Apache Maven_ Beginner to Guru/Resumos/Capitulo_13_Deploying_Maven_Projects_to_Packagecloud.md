# Capítulo 13: Deploying Maven Projects to Packagecloud

Neste capítulo, estudamos a última fase do ciclo de vida de compilação do Maven — o objetivo **`deploy`** —, aprendendo a publicar e compartilhar artefatos reutilizáveis (`.jar`, `.pom`, hashes SHA/MD5) em um repositório remoto na nuvem usando o serviço **Packagecloud**, além de dissecar o elemento `<distributionManagement>`, extensões do Maven Wagon e segregação de ambientes entre *Releases* e *Snapshots*.

---

## 1. Conceitos Fundamentais

### 1.1 A Fase `deploy` no Maven Lifecycle
Até aqui, os artefatos compilados residiam estritamente na máquina local (`target/` ou no cache local `~/.m2/repository` via `mvn install`).
* O objetivo da fase **`deploy`** é enviar o binário final empacotado para um repositório corporativo ou público remoto, disponibilizando a biblioteca para outros desenvolvedores e pipelines de integração contínua (CI/CD).
* O `mvn deploy` executa todo o ciclo anterior: `validate` ➔ `compile` ➔ `test` ➔ `package` ➔ `verify` ➔ `install` ➔ `deploy`.

---

### 1.2 Anatomia do `<distributionManagement>`
Para que o Maven saiba **onde** publicar o artefato gerado, declaramos o bloco `<distributionManagement>` no `pom.xml`. Ele exige a definição de dois destinos distintos:

```text
Projeto Maven (mvn deploy)
        │
        ├── A versão termina em "-SNAPSHOT"? (ex: 1.0-SNAPSHOT)
        │     └── SIM ──► Envia para <snapshotRepository>
        │
        └── A versão é final/estável? (ex: 1.0.0)
              └── NÃO ──► Envia para <repository> (Release)
```

1. **Repositório de Release (`<repository>`)**:
   * Destinado a versões estáveis e oficiais (ex: `1.0.0`).
   * **Imutabilidade**: Em repositórios de qualidade corporativa, uma vez publicada uma versão de release, ela nunca deve ser sobrescrita.
2. **Repositório de Snapshot (`<snapshotRepository>`)**:
   * Destinado a versões em desenvolvimento (ex: `1.0-SNAPSHOT`, `1.1-SNAPSHOT`).
   * **Mutabilidade via Timestamp**: Ao enviar múltiplos deploys de uma mesma versão SNAPSHOT, o repositório remoto registra um carimbo de data/hora no arquivo (ex: `app-1.1-20260913.140500-1.jar`), permitindo que os consumidores sempre baixem a versão mais recente do dia sem conflito de cache.

---

### 1.3 Extensões de Build e o Maven Wagon (`build extensions`)
Por padrão, o núcleo do Maven compreende protocolos tradicionais de transporte como HTTP/HTTPS e FTP.
* O **Packagecloud** utiliza uma camada personalizada de transporte via API.
* Para habilitar o Maven a conversar com os endpoints do Packagecloud, declara-se uma **extensão de build** (`<extension>`) com o artefato `io.packagecloud.maven.wagon:maven-packagecloud-wagon`.
* Isso introduz o prefixo de protocolo customizado: `packagecloud+https://`.

---

### 1.4 Autenticação via API Token no `settings.xml`
O upload de artefatos exige privilégios de escrita.
* No `pom.xml`, atribuem-se identificadores únicos (`<id>`) para cada repositório em `<distributionManagement>`.
* No arquivo `~/.m2/settings.xml`, criam-se blocos `<server>` com IDs **rigorosamente idênticos**, vinculando o **API Token** do Packagecloud no campo de `<password>`.

---

## 2. Sintaxe, Comandos & Configurações

### 2.1 Configuração do `pom.xml` (Extensão Wagon e `distributionManagement`)

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>guru.springframework</groupId>
    <artifactId>testing-project</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- Configuração do Destino de Deploy -->
    <distributionManagement>
        <repository>
            <id>packagecloud.release</id>
            <name>Packagecloud Releases</name>
            <url>packagecloud+https://packagecloud.io/springframeworkguru/release</url>
        </repository>
        <snapshotRepository>
            <id>packagecloud.snapshot</id>
            <name>Packagecloud Snapshots</name>
            <url>packagecloud+https://packagecloud.io/springframeworkguru/snapshot</url>
        </snapshotRepository>
    </distributionManagement>

    <build>
        <extensions>
            <!-- Provedor Maven Wagon para autenticação e upload no Packagecloud -->
            <extension>
                <groupId>io.packagecloud.maven.wagon</groupId>
                <artifactId>maven-packagecloud-wagon</artifactId>
                <version>0.0.6</version>
            </extension>
        </extensions>
    </build>
</project>
```

---

### 2.2 Configuração de Credenciais no `~/.m2/settings.xml`

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" ...>
    <servers>
        <!-- Credenciais para o Repositório de Releases -->
        <server>
            <id>packagecloud.release</id>
            <password>SEU_API_TOKEN_PACKAGECLOUD_AQUI</password>
        </server>

        <!-- Credenciais para o Repositório de Snapshots -->
        <server>
            <id>packagecloud.snapshot</id>
            <password>SEU_API_TOKEN_PACKAGECLOUD_AQUI</password>
        </server>
    </servers>
</settings>
```

> [!IMPORTANT]
> Os valores contidos na tag `<id>` do `settings.xml` (`packagecloud.release` e `packagecloud.snapshot`) devem corresponder exatamente aos valores `<id>` informados em `<distributionManagement>` no `pom.xml`. Se houver qualquer divergência de caractere (ex: singular vs. plural), o Maven emitirá erro de autenticação (`401 Unauthorized`).

---

## 3. Fluxo de Trabalho Prático de Releases

```bash
# 1. Fase de Desenvolvimento (publica em snapshot)
# pom.xml -> <version>1.0-SNAPSHOT</version>
mvn clean deploy

# 2. Publicação de Release Oficial (manual no curso)
# pom.xml -> Altera para <version>1.0</version>
mvn clean deploy

# 3. Abertura de Novo Ciclo de Desenvolvimento
# pom.xml -> Altera para <version>1.1-SNAPSHOT</version>
mvn clean deploy
```

---

## 4. Tabela de Comandos do Capítulo

| Comando | Descrição |
| :--- | :--- |
| `mvn clean deploy` | Executa o ciclo completo de build, testes, empacotamento e sobe o JAR para o repositório remoto correspondente. |
| `mvn deploy -DskipTests` | Realiza o deploy ignorando a execução da suíte de testes unitários. |
| `mvn help:effective-pom` | Inspeciona o POM efetivo para verificar as URLs configuradas em `distributionManagement`. |

---

## 5. Apêndice — Atualizações & Boas Práticas Modernas

### 5.1 Alternativas Modernas de Mercado ao Packagecloud
Embora o Packagecloud seja uma solução SaaS funcional para múltiplos tipos de pacotes (DEB, RPM, Ruby, Maven), no ecossistema moderno de desenvolvimento em nuvem destacam-se:

1. **GitHub Packages (Apache Maven Registry)**:
   * Totalmente integrado ao repositório de código e ao GitHub Actions.
   * Não requer criação de contas em plataformas terceiras; a autenticação no CI é feita automaticamente via segredo `${{ secrets.GITHUB_TOKEN }}`.
2. **GitLab Package Registry**:
   * Repositório Maven integrado por projeto ou por grupo no GitLab CI.
3. **Repositórios Nativos em Nuvem Corporativa**:
   * **AWS CodeArtifact**, **Google Cloud Artifact Registry** e **Azure Artifacts**: ideais para arquiteturas de nuvem sob rigorosas regras de governança e VPC privada.
4. **Nexus OSS / JFrog Artifactory**:
   * O padrão de ouro corporativo para infraestrutura de hospedagem própria (*on-premises* ou cluster em nuvem).

---

### 5.2 Deploy Atômico no `maven-deploy-plugin` (v3.x+)
Em versões antigas do Maven, em projetos multimódulos, o plugin de deploy enviava cada módulo assim que terminava sua compilação. Se o décimo módulo falhasse, os nove módulos anteriores já estavam publicados no repositório remoto, gerando um estado corrompido e inconsistente de release.
* **Boa Prática Moderna**: Usar `deployAtEnd`:
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-deploy-plugin</artifactId>
    <version>3.1.1</version>
    <configuration>
        <!-- Só envia os artefatos se todos os módulos do build passarem com sucesso -->
        <deployAtEnd>true</deployAtEnd>
    </configuration>
</plugin>
```

---

### 5.3 Automação de Releases: Adeus Edição Manual de Versões
No desenvolvimento moderno, editar manualmente o `pom.xml` para remover `-SNAPSHOT`, subir a tag no Git e incrementar para a próxima versão é considerado um anti-padrão sujeito a falha humana.
* **Ferramentas Modernas**:
  * **JReleaser**: Ferramenta moderna em alta na comunidade Java para automatizar changelogs, publicação no Maven Central, GitHub Packages, Homebrew e Docker.
  * **Semantic Release / Conventional Commits**: Calcula a nova versão (`1.0.0`, `1.1.0` ou `2.0.0`) automaticamente a partir das mensagens de commit Git (`feat:`, `fix:`, `feat!:`).
  * **CI/CD Triggers**: O deploy de releases finais ocorre exclusivamente através da criação de tags no Git (ex: `git tag v1.0.0 && git push origin v1.0.0`), executado por runners isolados e seguros de CI.
