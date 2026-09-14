# Capítulo 14: Deploying Maven Projects to Nexus

Neste capítulo, estudamos o **Sonatype Nexus Repository Manager (Nexus OSS)**, a solução padrão da indústria para gerenciamento corporativo de artefatos. Analisamos a diferença entre repositórios locais, proxies e grupos virtuais, configuramos o Maven para publicação segura de artefatos internos via `distributionManagement` e estabelecemos espelhos universais (`mirrors`) para centralizar todo o tráfego de dependências da organização.

---

## 1. Conceitos Fundamentais

### 1.1 O que é o Sonatype Nexus?
O **Nexus Repository OSS** é uma aplicação corporativa desenvolvida em Java pela **Sonatype** (a mesma organização responsável por manter o Maven Central).
* Enquanto serviços como o Packagecloud atuam puramente como repositórios remotos em nuvem, o Nexus funciona como um **gerenciador de repositórios completo**, permitindo hospedar artefatos próprios, intermediar o acesso à internet pública e consolidar regras de governança e segurança.
* Suporta múltiplos formatos: Maven 2, NPM, Docker/OCI, PyPI, RubyGems, NuGet, Helm, etc.

---

### 1.2 Os Três Tipos Essenciais de Repositórios no Nexus

```text
               ╔═══════════════════════════════════════════════╗
               ║          NEXUS REPOSITORY GROUP               ║
               ║               (nexus-group)                   ║
               ╚═══════════════════════════════════════════════╝
                                       │
        ┌──────────────────────────────┼──────────────────────────────┐
        ▼                              ▼                              ▼
┌─────────────────┐            ┌─────────────────┐            ┌─────────────────┐
│     HOSTED      │            │     HOSTED      │            │      PROXY      │
│  nexus-release  │            │ nexus-snapshot  │            │  maven-central  │
│ (Releases da    │            │ (Snapshots em   │            │ (Cache local do │
│  sua Empresa)   │            │  desenvolvimento)│           │  Maven Central) │
└─────────────────┘            └─────────────────┘            └─────────────────┘
        ▲                              ▲                              │
        │                              │                              ▼
  mvn deploy                     mvn deploy                     Internet Aberta
  (versão 1.0.0)                 (versão 1.0-SNAPSHOT)       (repo.maven.apache.org)
```

1. **Hosted Repository (Hospedado)**:
   * Armazena os artefatos compilados internamente pela sua equipe via `mvn deploy`.
   * **Releases**: Configurado com política de versão *Release* e `Disable redeploy` (garante imutabilidade dos artefatos oficiais).
   * **Snapshots**: Configurado com política de versão *Snapshot* (permite sobrescritas controladas por carimbo de data/hora).
2. **Proxy Repository (Intermediador/Cache)**:
   * Conecta-se a um repositório externo público (ex: Maven Central, Red Hat GA).
   * Na primeira requisição de um desenvolvedor, o Nexus baixa o JAR da internet, armazena no seu **Blob Store** interno e entrega ao desenvolvedor.
   * Nas próximas requisições (feitas por qualquer membro da empresa ou esteiras de CI/CD), o artefato é entregue imediatamente pela rede local, economizando largura de banda externa e acelerando os builds drasticamente.
3. **Group Repository (Agrupador Virtual)**:
   * Cria uma **URL única unificada** combinando múltiplos repositórios (releases internos, snapshots internos e o proxy do Maven Central).
   * Os desenvolvedores precisam configurar apenas este único endereço no `settings.xml` para resolver qualquer dependência do ecossistema.

---

### 1.3 Políticas de Versão e Layout
* **Version Policy**:
  * `Release`: Aceita estritamente artefatos sem o sufixo `-SNAPSHOT`.
  * `Snapshot`: Aceita estritamente versões terminadas em `-SNAPSHOT`.
  * `Mixed`: Aceita ambos (geralmente evitado em ambientes corporativos para não poluir o histórico de produção).
* **Layout Policy**:
  * `Strict`: Exige rigorosamente a convenção de diretórios do Maven (`/groupId/artifactId/version/...`).
  * `Permissive`: Tolera layouts alternativos (usado por ferramentas como SBT ou Gradle legado).
* **Deployment Policy**:
  * `Disable redeploy`: Bloqueia tentativas de publicar um artefato com a mesma versão de um já existente (indispensável para integridade de releases).

---

## 2. Sintaxe, Comandos & Configurações

### 2.1 Executando o Nexus localmente via Docker
```bash
# Executa o Nexus 3 em background mapeando a porta 8081
docker run -d -p 8081:8081 --name nexus sonatype/nexus3
```
* Acesso à interface Web: `http://localhost:8081`

---

### 2.2 Configurando Publicação no `pom.xml` (`distributionManagement`)
```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>guru.springframework</groupId>
    <artifactId>testing-project</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- Destinos de Deploy do Nexus -->
    <distributionManagement>
        <repository>
            <id>nexus-release</id>
            <name>Nexus Release Repository</name>
            <url>http://localhost:8081/repository/nexus-release/</url>
        </repository>
        <snapshotRepository>
            <id>nexus-snapshot</id>
            <name>Nexus Snapshot Repository</name>
            <url>http://localhost:8081/repository/nexus-snapshot/</url>
        </snapshotRepository>
    </distributionManagement>
</project>
```

---

### 2.3 Configurando Autenticação e Mirror Global em `~/.m2/settings.xml`

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" ...>
    <!-- 1. Credenciais de acesso para deploy no Nexus -->
    <servers>
        <server>
            <id>nexus-release</id>
            <username>admin</username>
            <password>admin123</password> <!-- Recomendado usar senha criptografada -->
        </server>
        <server>
            <id>nexus-snapshot</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
    </servers>

    <!-- 2. Espelhamento Universal: Redireciona todas as buscas para o Grupo Virtual do Nexus -->
    <mirrors>
        <mirror>
            <id>nexus-group-mirror</id>
            <name>Nexus Virtual Repository Group</name>
            <url>http://localhost:8081/repository/nexus-group/</url>
            <mirrorOf>*</mirrorOf> <!-- Intercepta Maven Central e qualquer outro repo -->
        </mirror>
    </mirrors>
</settings>
```

> [!TIP]
> Ao configurar `<mirrorOf>*</mirrorOf>` apontando para o `nexus-group`, você pode remover completamente blocos `<repositories>` redundantes dos seus arquivos `pom.xml`. Qualquer biblioteca (interna ou de terceiros) será intermediada e entregue pelo Nexus.

---

## 3. Comandos e Ciclo de Publicação

| Ação | Comando Maven | Comportamento no Nexus |
| :--- | :--- | :--- |
| **Deploy de Desenvolvimento** | `mvn clean deploy` *(com versão `1.0-SNAPSHOT`)* | Publica no repositório hosted `nexus-snapshot` gerando timestamp único. |
| **Deploy de Release** | `mvn clean deploy` *(com versão `1.0`)* | Publica no repositório hosted `nexus-release`. Bloqueia sobrescritas subsequentes. |
| **Resolução de Dependências** | `mvn clean package` | Busca no cache local; se ausente, solicita ao `nexus-group`, que serve do cache do Nexus ou baixa via proxy do Maven Central. |

---

## 4. Apêndice — Atualizações & Boas Práticas Modernas

### 4.1 Segurança de Inicialização no Nexus 3 Moderno
Em versões recentes do Sonatype Nexus 3 (a partir da versão 3.17+), a senha inicial fixa `admin123` foi abolida por motivos de segurança.
* Ao subir o contêiner pela primeira vez, o Nexus gera uma senha forte randômica armazenada no arquivo local:
  ```bash
  # Para recuperar a senha gerada no contêiner Docker:
  docker exec -it nexus cat /nexus-data/admin.password
  ```
* No primeiro login, a aplicação força o administrador a definir uma nova senha definitiva e a configurar as permissões de acesso anônimo (*Anonymous Access*).

### 4.2 Sonatype Nexus Firewall e Segurança de Dependências (SCA)
Em ambientes corporativos modernos, o Nexus atua como uma barreira ativa contra ataques à cadeia de suprimentos de software (*Software Supply Chain Security*):
* **Nexus Firewall / IQ Server**: Intercepta requisições de proxy e bloqueia em tempo real downloads de pacotes que contenham vulnerabilidades críticas (CVEs conhecidas), código malicioso (*typosquatting*) ou licenças jurídicas incompatíveis, impedindo que desenvolvedores tragam riscos para dentro da rede interna.

### 4.3 Políticas de Limpeza Automática (*Cleanup Policies*)
Em empresas com centenas de deploys diários de CI/CD, repositórios de SNAPSHOT podem facilmente consumir terabytes de armazenamento.
* O Nexus 3 introduziu **Cleanup Policies**: tarefas agendadas que removem automaticamente snapshots que não foram baixados ou atualizados nos últimos 30 ou 60 dias, mantendo o Blob Storage saudável.

### 4.4 Boa Prática para Mirrors: `external:*` vs `*`
Embora o curso utilize `<mirrorOf>*</mirrorOf>`, a recomendação moderna do Apache Maven é usar:
```xml
<mirrorOf>external:*</mirrorOf>
```
* **Vantagem**: O curinga `external:*` redireciona todas as requisições remotas para o Nexus, mas **não intercepta** repositórios locais em memória ou servidores HTTP fictícios executados em testes automatizados locais (`localhost` ou `file://`).
