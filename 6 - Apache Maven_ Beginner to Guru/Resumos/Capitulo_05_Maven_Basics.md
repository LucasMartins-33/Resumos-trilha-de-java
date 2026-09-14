# Capítulo 05: Maven Basics

Neste capítulo são estabelecidos os pilares conceituais, teóricos e arquiteturais do Apache Maven. O foco é compreender as engrenagens internas da ferramenta: identificação de artefatos por coordenadas, funcionamento dos repositórios, a camada de transporte Wagon, o modelo de objetos (POM), o algoritmo de resolução e escopos de dependências, a estrutura de diretórios padronizada, os ciclos de vida de build (*Build Lifecycles*), o Maven Wrapper e os arquétipos de projetos.

---

## 1. Coordenadas Maven (*Maven Coordinates*)

As coordenadas funcionam como o "endereço postal" absoluto de um artefato dentro do ecossistema Maven. Elas identificam unicamente qualquer biblioteca e definem sua localização física exata nos repositórios.

A tríade fundamental é conhecida como **GAV**:
1. **`groupId`:** Identifica a organização, empresa ou agrupamento do projeto.
   * *Convenção:* Domínio reverso da entidade (ex: `org.apache.commons`, `guru.springframework`, `com.google.guava`).
   * *Exceções históricas:* Bibliotecas pioneiras como `junit` utilizavam apenas `junit`.
2. **`artifactId`:** Nome do projeto/módulo específico (ex: `commons-lang3`, `spring-boot-starter-web`). Será a base do nome do arquivo `.jar` gerado.
3. **`version`:** Identificador da versão do artefato.

### 1.1. Esquema de Versionamento
O padrão de versões no Maven geralmente segue três números:
$$\text{Major} . \text{Minor} . \text{Incremental} \quad [ - \text{Qualifier} ]$$

* **Major (Maior):** Grandes quebras de compatibilidade ou reformulações de arquitetura (ex: `3.x.x`).
* **Minor (Menor):** Novas funcionalidades mantendo compatibilidade retroativa (ex: `x.2.x`).
* **Incremental (Patch/Bugfix):** Correções de bugs pontuais (ex: `x.x.1`).
* **Qualifier / Build:** Identificadores adicionais opcionais (ex: `RC1`, `RELEASE`, `M1`).

### 1.2. O Papel do `SNAPSHOT`
* **Release (versão sem `-SNAPSHOT`, ex: `1.0.0`):** Considerada **estável e imutável**. Uma vez baixada pelo Maven para o repositório local, ela **nunca mais é consultada ou baixada novamente** da internet.
* **SNAPSHOT (ex: `1.0.0-SNAPSHOT`):** Indica uma versão **em desenvolvimento ativo**.
  * Por padrão, o Maven conecta-se aos repositórios remotos uma vez ao dia (ou a cada build, se forçado via flag `-U`) para verificar se existem compilações mais recentes publicadas pela equipe.

---

## 2. Repositórios Maven (*Maven Repositories*)

Um repositório Maven é uma estrutura de diretórios padronizada para armazenamento e compartilhamento de artefatos (`.jar`, `.war`, `.pom`, `.sha1`, etc.).

```
                     ┌───────────────────────────┐
                     │     Repositório Local     │
                     │    (~/.m2/repository)     │
                     └─────────────┬─────────────┘
                                   │ (Se não encontrado)
                   ┌───────────────┴───────────────┐
                   ▼                               ▼
       ┌───────────────────────┐       ┌───────────────────────┐
       │     Maven Central     │       │ Repositórios Remotos  │
       │ (repo.maven.apache)   │       │ (Nexus, Artifactory,  │
       │                       │       │  JBoss, Google, etc.) │
       └───────────────────────┘       └───────────────────────┘
```

### 2.1. Tipos de Repositórios
1. **Local Repository (`~/.m2/repository`):**
   * Diretório local na máquina do desenvolvedor.
   * Funciona como cache de tudo que foi baixado de repositórios remotos e recebe artefatos locais compilados com `mvn install`.
   * Estrutura de pastas segue as coordenadas: `groupId` vira caminho de pastas (substituindo pontos por barras `/`), seguido de `artifactId`, `version` e o arquivo.
   * Exemplo: `org.apache.commons:commons-lang3:3.8.1` é gravado em:
     `~/.m2/repository/org/apache/commons/commons-lang3/3.8.1/`
2. **Central Repository (Maven Central):**
   * Repositório público global mantido pela comunidade Maven/Sonatype com mais de uma dezena de milhões de artefatos. Configurado como padrão no Maven.
3. **Remote Repositories (Privados/Corporativos):**
   * Servidores como **Sonatype Nexus** ou **JFrog Artifactory** mantidos em redes corporativas para armazenar bibliotecas proprietárias da empresa e atuar como proxy do Maven Central.

### 2.2. O que é armazenado na pasta da versão:
* `*.jar`: O binário compilado.
* `*.pom`: O descritor do artefato (fundamental para que o Maven descubra as dependências transitivas dessa biblioteca).
* `*.sha1` / `*.md5`: Hashes de integridade criptográfica para validação de download.
* `*-sources.jar` e `*-javadoc.jar`: Código-fonte e documentação (quando solicitados).

---

## 3. Maven Wagon

O **Maven Wagon** é uma camada de abstração de transporte do Apache Maven. Ele unifica o protocolo de rede usado para baixar e publicar artefatos de e para repositórios.

* **Provedores Suportados:** HTTP, HTTPS, File (sistema de arquivos local), FTP, SSH/SCP, WebDAV e SCM.
* **Benefício:** O núcleo do Maven apenas solicita o arquivo; o Wagon cuida dos detalhes do protocolo de transporte subjacente de forma transparente.
* **Uso Típico:** Configurações de autenticação corporativa (usuário/senha) e proxies corporativos no arquivo `settings.xml`.

---

## 4. O POM e o POM Efetivo (*Effective POM*)

O `pom.xml` (*Project Object Model*) é um documento XML rigorosamente validado pelo schema `maven-4.0.0.xsd`.

### 4.1. Herança e Super POM
O Maven possui um **Super POM** embutido em suas bibliotecas internas que define configurações e valores padrão para qualquer projeto (como diretórios padrão, repositório central e versões base de plugins).

### 4.2. O POM Efetivo (*Effective POM*)
O **Effective POM** é a visão consolidada final do projeto em tempo de execução:
$$\text{Super POM (Padrões do Maven)} + \text{Parent POMs herdados} + \text{Seu pom.xml local} = \text{Effective POM}$$

* **Comando para inspecionar o POM Efetivo via terminal:**
  ```bash
  mvn help:effective-pom
  ```
  *(Exibe todos os plugins, diretórios e repositórios atribuídos por baixo dos panos pelo Maven)*.

---

## 5. Dependências no Maven

O gerenciamento automático de dependências é uma das maiores fortalezas do Maven.

### 5.1. Conceitos Fundamentais
* **Dependência Transitiva:** Se seu projeto depende de **A**, e **A** depende de **B**, o Maven adiciona **B** automaticamente ao seu projeto.
* **Dependências Cíclicas:** A depende de B e B depende de A. O Maven detecta isso e aborta o build com erro fatal.
* **Dependências Opcionais (`<optional>true</optional>`):** Marcadas para que projetos a jusante (*downstream*) não as herdem automaticamente.
* **Exclusões (`<exclusions>`):** Permitem remover manualmente uma dependência transitiva indesejada ou conflitante.

### 5.2. Mediação de Dependências (*Dependency Mediation*)
Quando duas bibliotecas transitivas exigem versões diferentes da mesma biblioteca de terceiro, o Maven aplica a regra da **Definição Mais Próxima** (*Nearest Definition Wins*):
* A versão localizada no nível mais próximo da raiz da árvore de dependências será a escolhida.
* Se duas versões estiverem na mesma profundidade, **a primeira declarada no POM vence**.

### 5.3. Escopos de Dependência (*Dependency Scopes*)

O escopo define em quais classpaths a dependência estará presente e se ela se propagará para projetos dependentes:

| Escopo | Compile Classpath | Test Classpath | Runtime Classpath | Empacotado no JAR/WAR? | É Transitiva? | Exemplo Típico |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **`compile`** (Padrão) | Sim | Sim | Sim | Sim | Sim | `commons-lang3`, `guava` |
| **`provided`** | Sim | Sim | Não | Não | Não | `servlet-api`, Lombok |
| **`runtime`** | Não | Sim | Sim | Sim | Sim | Drivers JDBC (`postgresql`), implementações de logging |
| **`test`** | Não | Sim | Não | Não | Não | `junit-jupiter`, `mockito-core` |
| **`system`** *(legado)* | Sim | Sim | Não | Não | Não | JAR local indicado por caminho absoluto via `<systemPath>` |
| **`import`** | N/A | N/A | N/A | N/A | N/A | Utilizado em `<dependencyManagement>` para importar BOMs |

> [!WARNING]
> **Erro comum com testes:** Esquecer a tag `<scope>test</scope>` em bibliotecas como JUnit ou Mockito fará com que elas sejam classificadas como `compile`, forçando todos os clientes que consumirem sua biblioteca a baixarem o JUnit desnecessariamente.

### 5.4. Plugin de Dependências (`maven-dependency-plugin`)

Comandos utilitários essenciais:
```bash
# Exibe a árvore visual completa de dependências diretas e transitivas
mvn dependency:tree

# Baixa todas as dependências do projeto para permitir builds offline
mvn dependency:go-offline

# Limpa o cache do repositório local referente ao projeto atual
mvn dependency:purge-local-repository

# Faz o download dos arquivos JAR de código-fonte das dependências
mvn dependency:sources
```

---

## 6. Estrutura Padrão de Diretórios (*Standard Directory Layout*)

O Maven segue rigidamente o padrão de **Convenção sobre Configuração**:

| Diretório | Finalidade |
| :--- | :--- |
| `src/main/java` | Código-fonte de produção Java (`.java`) |
| `src/main/resources` | Arquivos de configuração de produção (properties, YAML, XML) |
| `src/main/webapp` | Recursos de aplicações web tradicionais WAR (HTML, CSS, JSP) *(legado)* |
| `src/test/java` | Classes de testes unitários e de integração |
| `src/test/resources` | Recursos e configurações exclusivas de teste |
| `src/it` | Testes de integração de plugins Maven |
| `src/site` | Arquivos descritores para geração de website com Maven Site |
| `target` | Diretório temporário gerado pelo Maven contendo bytecodes e pacotes finais |

---

## 7. Ciclos de Vida de Build (*Build Lifecycles*)

Um ciclo de vida de build no Maven é uma sequência ordenada de **Fases** (*Phases*). O Maven possui **3 ciclos de vida nativos**:

### 7.1. Os Três Ciclos Nativos
1. **`clean`:** Limpeza do projeto e exclusão dos artefatos de compilações anteriores (`target/`).
2. **`default`:** Ciclo central de compilação, testes, empacotamento e distribuição do software.
3. **`site`:** Geração de documentação e website HTML do projeto.

### 7.2. Fases vs Metas de Plugins (*Phases vs Goals*)
* **Fase (*Phase*):** Um estágio conceitual abstrato do ciclo de vida (ex: `compile`, `test`, `package`).
* **Meta de Plugin (*Plugin Goal*):** A tarefa real e concreta de código que realiza o trabalho (ex: `compiler:compile`, `surefire:test`, `jar:jar`).
* O Maven mapeia metas de plugins para fases específicas. Quando você executa `mvn compile`, o Maven executa todas as fases que antecedem `compile` em ordem cronológica.

### 7.3. As Fases Principais do Ciclo `default`
1. **`validate`:** Valida a integridade do POM e a estrutura do projeto.
2. **`compile`:** Compila o código-fonte de produção (`src/main/java`).
3. **`test`:** Executa testes unitários usando o framework de testes configurado (via Surefire Plugin).
4. **`package`:** Empacota o código compilado no formato declarado (JAR, WAR, etc.).
5. **`verify`:** Executa verificações de qualidade e testes de integração (Failsafe Plugin).
6. **`install`:** Copia o pacote final para o repositório local (`~/.m2/repository`), disponibilizando-o para outros projetos locais na máquina.
7. **`deploy`:** Copia o artefato final para o repositório corporativo remoto (Nexus/Artifactory/Maven Central) para compartilhamento global.

---

## 8. Maven Wrapper (`mvnw`)

O **Maven Wrapper** é uma ferramenta para empacotar o Maven junto ao código do projeto, garantindo que qualquer desenvolvedor ou servidor de CI compile o projeto com a **versão exata e homologada do Maven**, sem requerer instalação prévia no sistema operacional.

### 8.1. Arquivos que compõem o Wrapper
* `mvnw`: Script executável Bash para Linux e macOS.
* `mvnw.cmd`: Script batch para Windows.
* `.mvn/wrapper/maven-wrapper.properties`: Contém a URL e versão exata do Maven a ser baixada.
* `.mvn/wrapper/maven-wrapper.jar`: Executável leve que efetua o download do Maven na primeira execução.

---

## 9. Arquétipos Maven (*Archetypes*)

Um **Archetype** é um template ou esqueleto de projeto gerado a partir de parâmetros.
* **Comando para gerar:**
  ```bash
  mvn archetype:generate
  ```
  *(Apresenta um menu interativo com centenas de modelos de projetos)*.
* **Exemplos clássicos:** `maven-archetype-quickstart` (projeto básico Java SE), `maven-archetype-webapp` (aplicação web WAR).

---

## 10. Apêndice — Atualizações & Boas Práticas Modernas

### 1. Novo Comando Oficial do Maven Wrapper (Substituindo o Takari)
Na transcrição histórica da aula, o instrutor utiliza o plugin legado da Takari (`io.takari:maven:wrapper`).
* **Hoje:** O wrapper foi incorporado oficialmente ao Apache Maven como o plugin oficial `maven-wrapper-plugin`.
* **Comando atual recomendado:**
  ```bash
  mvn wrapper:wrapper -Dmaven=3.9.6
  ```

### 2. Maven Central: Bloqueio Total de HTTP (Obrigatoriedade de HTTPS)
* Desde janeiro de 2020, o **Maven Central baniu completamente conexões HTTP não criptografadas**. Qualquer URL de repositório configurada como `http://repo.maven.apache.org` falhará com erro `501 HTTPS Required`.
* Hoje em dia, além de HTTPS, todos os artefatos enviados ao Maven Central exigem assinaturas digitais GPG válidas (`.asc`) e checksums SHA-256 / SHA-512.

### 3. Depreciação Formal do Escopo `system`
* O escopo `<scope>system</scope>` foi formalmente depreciado e será descontinuado no Maven 4.
* **Alternativa moderna:** Caso possua um JAR legado sem repositório, instale-o no repositório local usando:
  ```bash
  mvn install:install-file -Dfile=minha-lib.jar -DgroupId=com.empresa -DartifactId=minha-lib -Dversion=1.0 -Dpackaging=jar
  ```
  Ou utilize um repositório corporativo interno (Nexus/Artifactory) ou o repositório de pacotes do GitHub/GitLab.

### 4. Riscos da Regra *"Nearest Wins"* e o `maven-enforcer-plugin`
A regra de resolução "a definição mais próxima vence" do Maven pode causar *downgrades* silenciosos em tempo de execução (uma dependência transitiva antiga sobrescreve uma nova porque está um nível mais perto da raiz).
* **Solução moderna:** Em projetos corporativos, utiliza-se o plugin **`maven-enforcer-plugin`** com a regra `<dependencyConvergence/>` para forçar o build a falhar caso haja qualquer versão conflitante não resolvida explicitamente via `<dependencyManagement>`.

### 5. Arquétipos Antigos vs Geradores Modernos
Os arquétipos clássicos do Maven (`maven-archetype-quickstart`) ficaram defasados por anos (vinham com Java 1.5 e JUnit 3.8.1). No cenário moderno, projetos não utilizam mais `mvn archetype:generate`. Em vez disso, usam portais geradores com stacks atualizadas:
* **Spring Boot:** [start.spring.io](https://start.spring.io) (Spring Initializr)
* **Quarkus:** [code.quarkus.io](https://code.quarkus.io)
* **Micronaut:** [launch.micronaut.io](https://launch.micronaut.io)
