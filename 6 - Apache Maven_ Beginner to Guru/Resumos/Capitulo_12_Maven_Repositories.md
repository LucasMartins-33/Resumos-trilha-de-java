# Capítulo 12: Maven Repositories

Neste capítulo, examinamos a fundo o funcionamento dos **Repositórios Maven** (*Maven Repositories*), a ordem de resolução de artefatos, o uso e anatomia dos arquivos `settings.xml` e `settings-security.xml`, o espelhamento de repositórios (*mirrors*), a instalação manual de dependências e a autenticação segura com criptografia de senhas para repositórios remotos privados.

---

## 1. Conceitos Fundamentais

### 1.1 O Ecossistema de Repositórios e Fluxo de Resolução
O Maven adota uma estratégia escalonada para localizar dependências e plugins:

```text
[Compilação do Projeto]
         │
         ▼
 1. Cache Local (~/.m2/repository)
    Encontrado? ── SIM ──► Utiliza o artefato local
         │ NÃO
         ▼
 2. Repositórios Remotos Declarados no POM / Settings
    (Busca no Maven Central e em repositórios adicionais por ordem de declaração)
         │
         ├── Encontrado ──► Baixa no ~/.m2/repository ──► Utiliza na compilação
         │
         └── Não Encontrado ──► Falha do Build (ArtifactResolutionException)
```

1. **Repositório Local (`~/.m2/repository`)**: Atua como cache em disco na máquina do desenvolvedor. Se um JAR já foi baixado em qualquer projeto anterior, o Maven evita tráfego de rede e reutiliza o arquivo local.
2. **Repositório Remoto Central (Maven Central)**: Maior repositório público do mundo, mantido pela **Sonatype**, com mais de 10 milhões de versões indexadas.
3. **Repositórios Remotos Adicionais**: Configurados via `pom.xml` ou `settings.xml` para artefatos específicos (ex: Red Hat/JBoss, Spring Milestones/Snapshots, Atlassian, repositórios corporativos internos).

---

### 1.2 Anatomia do Arquivo `settings.xml`
Enquanto o `pom.xml` descreve o **projeto**, o `settings.xml` descreve o **ambiente de execução do usuário ou da máquina**.

* **User Settings**: `${user.home}/.m2/settings.xml` (específico do usuário logado na máquina).
* **Global Settings**: `${maven.home}/conf/settings.xml` (aplica-se a todos os usuários da máquina/servidor).
* **Sobrescrita via CLI**:
  * `-s /caminho/meu-settings.xml` (sobrescreve user settings — muito usado em pipelines de CI/CD).
  * `-gs /caminho/global-settings.xml` (sobrescreve global settings).

#### Principais Elementos do `settings.xml`:
* `<localRepository>`: Redireciona a pasta de cache local para outro disco/diretório.
* `<interactiveMode>`: Habilita/desabilita prompts interativos do Maven no terminal (`true`/`false`).
* `<offline>`: Força o Maven a operar sem conexão de rede (utiliza estritamente o que já está em cache).
* `<proxies>`: Configuração de proxy HTTP/HTTPS corporativo com autenticação.
* `<servers>`: Armazena credenciais (usuário, senha, chave privada) de servidores e repositórios remotos.
* `<mirrors>`: Define redirecionamentos de repositórios (ex: espelhos geográficos ou proxy corporativo).
* `<profiles>` / `<activeProfiles>`: Declaração e ativação global de perfis para todos os builds locais.

---

### 1.3 Espelhos de Repositório (*Repository Mirrors*)
Um mirror substitui as requisições destinadas a um determinado repositório por outro endereço.

* **Casos de Uso**:
  * **Otimização Geográfica**: Apontar o Maven Central para um espelho mais próximo do desenvolvedor.
  * **Controle Corporativo**: Forçar que todas as requisições de bibliotecas passem obrigatoriamente por um gerenciador de repositórios interno da empresa (Nexus, Artifactory), substituindo a internet aberta (`<mirrorOf>*</mirrorOf>`).

---

### 1.4 Repositórios Públicos e Especializados

| Repositório | Mantenedor / URL Base | Finalidade |
| :--- | :--- | :--- |
| **Maven Central** | Sonatype (`repo.maven.apache.org/maven2`) | Padrão da indústria; repositório primário de código aberto Java. |
| **Red Hat / JBoss** | Red Hat (`maven.repository.redhat.com/ga/`) | Artefatos corporativos Red Hat, WildFly, EJB, Hibernate e extensões Quarkus. |
| **Spring Snapshots / Milestones** | Broadcom / Spring (`repo.spring.io/snapshot`, `milestone`) | Acesso a versões preliminares e correções de bugs antes do lançamento no Central. |
| **Atlassian Public** | Atlassian (`packages.atlassian.com/mvn/`) | Dependências para desenvolvimento de plugins Jira, Confluence, Bitbucket. |
| **Oracle Maven Repository** | Oracle (`maven.oracle.com`) | Repositório autenticado com produtos Oracle, WebLogic e drivers de banco legados. |

---

### 1.5 Instalação Manual no Cache Local: `install-file`
Quando uma biblioteca de terceiros ou legada não existe em nenhum repositório público (ou restrições legais impedem sua distribuição pública), o Maven permite injetá-la diretamente no repositório local:

```bash
mvn install:install-file \
  -Dfile=caminho/para/biblioteca.jar \
  -DgroupId=com.empresa \
  -DartifactId=biblioteca-legada \
  -Dversion=1.0.0 \
  -Dpackaging=jar
```

> [!WARNING]
> Instalar manualmente via `install-file` resolve a compilação apenas na máquina local. Em servidores de CI/CD ou na máquina de outros membros da equipe, o build falhará a menos que o comando seja repetido em cada máquina ou que o artefato seja publicado em um repositório corporativo compartilhado (Nexus/Artifactory).

---

### 1.6 Segurança e Criptografia de Credenciais
Nunca armazene senhas em texto puro (*plain text*) dentro do `settings.xml`. O Maven possui um subsistema criptográfico integrado em dois níveis:

```text
1. Senha Mestra (Master Password)
   mvn --encrypt-master-password
   Armazenada em: ~/.m2/settings-security.xml

2. Senha do Servidor/Repositório
   mvn --encrypt-password
   (Criptografa a senha usando a chave mestra)
   Armazenada em: ~/.m2/settings.xml (<server><password>)
```

* **Vínculo por ID**: A propriedade `<id>` configurada em `<server>` dentro do `settings.xml` **deve ser idêntica** ao `<id>` da tag `<repository>` no `pom.xml` para que o Maven injete as credenciais automaticamente no momento do download.

---

## 2. Sintaxe, Comandos & Configurações

### 2.1 Declarando um Repositório Remoto no `pom.xml`
```xml
<repositories>
    <repository>
        <id>redhat-ga</id>
        <name>Red Hat General Availability Repository</name>
        <url>https://maven.repository.redhat.com/ga/</url>
        <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
        </releases>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>
```

#### Políticas de Atualização e Verificação:
* `<updatePolicy>`: `always` (toda execução), `daily` (padrão, uma vez ao dia), `interval:X` (a cada X minutos) ou `never`.
* `<checksumPolicy>`: `fail` (aborta o build se o hash SHA/MD5 for inválido), `warn` (emite aviso no console), `ignore`.

---

### 2.2 Configurando Espelho (*Mirror*) no `settings.xml`
```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
                              http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <mirrors>
        <mirror>
            <id>uk-central-mirror</id>
            <name>Central Repository UK Mirror</name>
            <url>https://uk.maven.org/maven2</url>
            <mirrorOf>central</mirrorOf>
        </mirror>
    </mirrors>
</settings>
```
* `<mirrorOf>central</mirrorOf>`: Intercepta e redireciona apenas o repositório Maven Central.
* `<mirrorOf>*</mirrorOf>`: Intercepta qualquer repositório (comum em redes corporativas).

---

### 2.3 Repositório Global via Profile em `settings.xml`
Quando um repositório precisa estar disponível para **todos** os projetos da máquina sem poluir os arquivos `pom.xml`:

```xml
<settings>
    <profiles>
        <profile>
            <id>jboss-profile</id>
            <repositories>
                <repository>
                    <id>redhat-ga</id>
                    <url>https://maven.repository.redhat.com/ga/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                </repository>
            </repositories>
        </profile>
    </profiles>

    <!-- Ativa o perfil permanentemente para todas as execuções locais -->
    <activeProfiles>
        <activeProfile>jboss-profile</activeProfile>
    </activeProfiles>
</settings>
```

---

### 2.4 Criptografia Completa de Senhas de Acesso

#### Passo 1: Gerar e salvar a Senha Mestra
Execute no terminal (sem passar a senha como argumento para evitar histórico no bash):
```bash
mvn --encrypt-master-password
# Digite a senha mestra quando solicitado
# Retorna: {jSMOWnoPFgs734mF9B03G8k+m+a78pG1}
```

Crie o arquivo `~/.m2/settings-security.xml`:
```xml
<settingsSecurity>
    <master>{jSMOWnoPFgs734mF9B03G8k+m+a78pG1}</master>
</settingsSecurity>
```

#### Passo 2: Criptografar a senha do repositório
```bash
mvn --encrypt-password
# Digite a senha do repositório (ex: sua senha do Oracle OTN ou Nexus)
# Retorna: {COQLCE6DU6GvuS5SEvfYVsQ=}
```

#### Passo 3: Configurar o servidor autenticado no `~/.m2/settings.xml`
```xml
<settings>
    <servers>
        <server>
            <id>maven.oracle.com</id>
            <username>usuario@email.com</username>
            <password>{COQLCE6DU6GvuS5SEvfYVsQ=}</password>
            <configuration>
                <basicAuthScope>
                    <host>ANY</host>
                    <port>ANY</port>
                    <realm>OAM 11g</realm>
                </basicAuthScope>
                <httpConfiguration>
                    <all>
                        <params>
                            <property>
                                <name>http.protocol.allow-circular-redirects</name>
                                <value>%b,true</value>
                            </property>
                        </params>
                    </all>
                </httpConfiguration>
            </configuration>
        </server>
    </servers>
</settings>
```

---

## 3. Tabela de Comandos do Capítulo

| Comando | Função Principal |
| :--- | :--- |
| `mvn --encrypt-master-password` | Gera a hash da chave de segurança mestra para o `settings-security.xml`. |
| `mvn --encrypt-password` | Criptografa uma senha de repositório utilizando a chave mestra configurada. |
| `mvn install:install-file ...` | Injeta manualmente um arquivo JAR no repositório local `~/.m2/repository`. |
| `mvn help:effective-pom` | Mostra todos os repositórios resolvidos e mesclados (POM + Settings). |
| `mvn help:effective-settings` | Exibe as configurações mescladas do usuário e globais do Maven. |
| `mvn clean package -s /caminho/ci-settings.xml` | Executa o build utilizando um arquivo `settings.xml` específico. |
| `mvn clean package -o` | Executa o build no modo offline (sem verificar repositórios remotos). |

---

## 4. Apêndice — Atualizações & Boas Práticas Modernas

### 4.1 Fim do JCenter e Bintray
Durante anos, o **JCenter** (da JFrog) foi um dos maiores repositórios alternativos do mundo e o repositório padrão do Gradle/Android Studio. 
* **Fevereiro de 2021**: A JFrog anunciou a descontinuação e pôs o JCenter em modo somente-leitura, sendo desativado permanentemente.
* **Impacto**: Qualquer projeto que ainda referencie `jcenter.bintray.com` sofrerá falhas de build. O padrão universal de mercado consolidou-se no **Maven Central** e no **Google Maven Repository** (`maven.google.com`).

### 4.2 Modernização do Oracle JDBC no Maven Central
No período em que o curso original foi gravado, a Oracle impunha termos contratuais rígidos (*Click-Through License*) que proibiam a distribuição pública do driver `ojdbc`, obrigando desenvolvedores a baixar manualmente os JARs ou registrar contas no OTN.
* **Nova Realidade (FUTC License)**: Desde setembro de 2019 (a partir da versão 19.3+), a Oracle mudou sua política de licenciamento.
* **Disponibilidade Direta**: Os drivers oficiais do Oracle Database estão **oficialmente disponíveis no Maven Central**, livres de autenticação ou registro:
```xml
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc11</artifactId> <!-- Para Java 11, 17 e 21 -->
    <version>23.3.0.23.09</version>
</dependency>
```

### 4.3 Padrão Corporativo Moderno: Nexus e Artifactory
Em organizações profissionais, desenvolvedores **não** acessam o Maven Central diretamente. É padrão arquitetural implantar um gerenciador de repositórios corporativo (**Sonatype Nexus Repository Manager** ou **JFrog Artifactory**):
1. **Cache Local de Rede**: Reduz o consumo de banda de internet na empresa.
2. **Escaneamento Contínuo de Segurança (SCA)**: Ferramentas como *Sonatype Nexus Firewall* e *JFrog Xray* bloqueiam o download de dependências públicas com vulnerabilidades conhecidas (CVEs / Log4Shell) ou licenças agressivas (GPL em software proprietário).
3. **Distribuição Interna**: Fornece repositório para hospedagem e compartilhamento de artefatos SNAPSHOT e RELEASE desenvolvidos pelos times internos.

### 4.4 Gerenciamento Seguro de Credenciais em CI/CD
Embora o `settings-security.xml` resolva o problema do texto puro em desktops locais, em pipelines de CI/CD (GitHub Actions, GitLab CI, Jenkins) a melhor prática moderna é:
* **Interpolação de Variáveis de Ambiente no `settings.xml`**:
```xml
<server>
    <id>internal-nexus</id>
    <username>${env.CI_REGISTRY_USER}</username>
    <password>${env.CI_REGISTRY_PASSWORD}</password>
</server>
```
* As credenciais são injetadas dinamicamente em tempo de execução via segredos mascarados (*Secrets/Vault*), garantindo que nenhum hash ou credencial persista em arquivos de configuração estáticos.
