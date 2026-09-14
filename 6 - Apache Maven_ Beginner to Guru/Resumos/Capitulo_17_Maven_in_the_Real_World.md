# Capítulo 17: Maven in the Real World

Neste capítulo prático e avançado, acompanhamos um caso de uso real de engenharia de software envolvendo múltiplos projetos de código aberto interdependentes. Diagnosticamos conflitos graves de classpath (*Classloader Trap* e `NoSuchMethodError`), dissecamos a árvore de dependências do Maven, exploramos o fluxo de desenvolvimento com versões **SNAPSHOT** entre projetos locais e realizamos o ciclo completo de auditoria e publicação de uma biblioteca no **Maven Central** via Sonatype OSSRH.

---

## 1. Conceitos Fundamentais

### 1.1 O Ecossistema do Estudo de Caso
O cenário envolve quatro projetos distribuídos:

```text
┌────────────────────────────────────────────────────────────────────────┐
│  spring-cloud-contract-oa3 (Projeto do Autor)                          │
│  Extensão para suporte a OpenAPI 3 em contratos Spring Cloud Contract   │
└────────────────────────────────────────────────────────────────────────┘
        ▲                                                ▲
        │ Dependência                                    │ Dependência
        │                                                │
┌───────────────────────┐                        ┌───────────────────────┐
│ fraud-service-scc     │                        │ payor-service         │
│ (Microsserviço de     │                        │ (Microsserviço que    │
│  Exemplo de Fraude)   │                        │  usa validação dupla) │
└───────────────────────┘                        └───────────────────────┘
                                                         │
                                                         ▼
                                       ┌──────────────────────────────────┐
                                       │ Atlassian Swagger Validator      │
                                       └──────────────────────────────────┘
```

---

### 1.2 Anatomia de um Conflito Real: O "Classloader Trap"
O microsserviço `payor-service` quebrou em tempo de execução com o erro fatal:
```text
java.lang.NoSuchMethodError: io.swagger.parser.OpenAPIParser.readLocation(...)
```

#### Como o conflito aconteceu?
1. **O Projeto do Autor (`spring-cloud-contract-oa3`)**: Utilizava uma versão preliminar antiga do parser OpenAPI:
   * Coordenadas: `io.swagger:swagger-parser:2.0.0-rc1`
2. **A Dependência da Atlassian (`swagger-request-validator`)**: Utilizava a versão estável mais nova do parser:
   * Coordenadas: `io.swagger.parser.v3:swagger-parser:2.0.5`
3. **A Falha de Mediação do Maven**:
   * O mecanismo padrão do Maven (*Nearest Wins*) só compara versões se o `groupId` e o `artifactId` forem idênticos.
   * Como a equipe da OpenAPI mudou o groupId/artifactId inserindo `.v3` nas coordenadas, o Maven assumiu que eram duas bibliotecas totalmente diferentes e incluiu **ambos os JARs no Classpath**.
4. **O Colapso da JVM**:
   * Ambos os JARs possuíam classes com nomes de pacote idênticos (`io.swagger.parser.OpenAPIParser`).
   * O Classloader da JVM carregou a primeira classe que encontrou no disco (a versão antiga RC1).
   * Quando o validador da Atlassian tentou chamar o método `readLocation(...)` (que só existia na versão 2.0.5), o método não existia na classe carregada em memória, explodindo em `NoSuchMethodError`.

---

### 1.3 Investigação com `mvn dependency:tree`
A árvore de dependências é a ferramenta primária para desmascarar versões duplicadas e identificar de onde uma biblioteca indesejada está sendo puxada:
```bash
mvn dependency:tree > tree.txt
```
Ao inspecionar o arquivo gerado, localizam-se os galhos conflitantes e a divergência de versões transitivas.

---

### 1.4 Ciclo de Desenvolvimento Interprojetos com SNAPSHOTs
Ao desenvolver uma biblioteca (`oa3`) e testá-la simultaneamente em aplicações consumidoras (`payor-service`):
1. **Na Biblioteca**:
   * Mantém-se a versão em `2.0.2.BUILD-SNAPSHOT`.
   * Executa-se `mvn clean install` para compilar e registrar o SNAPSHOT no repositório local `~/.m2/repository`.
2. **Nos Projetos Consumidores**:
   * Aponta-se a dependência para a versão `BUILD-SNAPSHOT`.
   * O Maven detecta a versão local recém-instalada no `.m2` e compila sem necessidade de publicação remota.
3. **A Flag `-U` (`--update-snapshots`)**:
   * Quando a equipe trabalha em máquinas separadas com repositório remoto compartilhado, o Maven por padrão só checa atualizações de snapshots **uma vez por dia**.
   * Para forçar o download imediato do snapshot mais recente gerado por um colega, utiliza-se:
     ```bash
     mvn clean package -U
     ```

---

### 1.5 Armadilhas do Maven Release Plugin em Projetos Complexos
Durante a publicação da release para o Maven Central, surgiram três obstáculos clássicos de engenharia:
1. **Snapshots Upstream Vazando no Build**: O plugin aborta o `release:prepare` se encontrar dependências em SNAPSHOT vindas de pais upstream.
2. **Propriedade `${project.version}` em Módulos Importados**: O Spring Cloud Contract utilizava `${project.version}` em seus BOMs, fazendo com que o projeto filho tentasse resolver dependências na versão do filho em vez da versão do framework pai.
3. **A Correção**: Forçar a importação correta declarando a versão explicitamente dentro de `<dependencyManagement>`:
   ```xml
   <dependencyManagement>
       <dependencies>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-contract-dependencies</artifactId>
               <version>${spring.cloud.contract.version}</version>
               <type>pom</type>
               <scope>import</scope>
           </dependency>
       </dependencies>
   </dependencyManagement>
   ```

---

### 1.6 O Processo de Publicação no Maven Central (Sonatype OSSRH)
O Maven Central não permite uploads diretos e desprotegidos. O fluxo exige:
1. **Área de Staging Temporária (`oss.sonatype.org`)**: Os artefatos são enviados para um repositório fechado de staging exclusivo do usuário.
2. **Regras Estritas de Validação do Maven Central**:
   * Código compilado (`.jar`).
   * Código-fonte anexado (`-sources.jar` via `maven-source-plugin`).
   * Documentação da API (`-javadoc.jar` via `maven-javadoc-plugin`).
   * Assinatura criptográfica PGP/GPG de todos os arquivos (`.asc` via `maven-gpg-plugin`).
   * Metadados completos no `pom.xml`: `<name>`, `<description>`, `<url>`, `<licenses>`, `<developers>` e `<scm>`.
3. **Fechamento e Release (`Close & Release`)**:
   * A Sonatype valida automaticamente todos os checksums e assinaturas.
   * Ao fechar o repositório de staging com sucesso, os artefatos são sincronizados com os espelhos mundiais do Maven Central.

---

## 2. Sintaxe, Comandos & Configurações

### 2.1 Plugins Obrigatórios para Publicação no Maven Central
Configuração necessária no `pom.xml` para cumprir as regras do Sonatype OSSRH:

```xml
<build>
    <plugins>
        <!-- 1. Geração de Sources JAR -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>3.3.0</version>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <goals>
                        <goal>jar-no-fork</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- 2. Geração de Javadoc JAR -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-javadoc-plugin</artifactId>
            <version>3.6.3</version>
            <executions>
                <execution>
                    <id>attach-javadocs</id>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>

        <!-- 3. Assinatura Criptográfica com GPG -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-gpg-plugin</artifactId>
            <version>3.2.0</version>
            <executions>
                <execution>
                    <id>sign-artifacts</id>
                    <phase>verify</phase>
                    <goals>
                        <goal>sign</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

---

### 2.2 Configuração do Repositório de Staging da Sonatype
```xml
<distributionManagement>
    <repository>
        <id>ossrh</id>
        <name>Central Directory Free Staging Repository</name>
        <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
    </repository>
    <snapshotRepository>
        <id>ossrh</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots</url>
    </snapshotRepository>
</distributionManagement>
```

---

## 3. Tabela de Comandos de Resolução e Diagnóstico

| Comando | Finalidade |
| :--- | :--- |
| `mvn dependency:tree` | Exibe a hierarquia de dependências diretas e transitivas do projeto. |
| `mvn dependency:tree -Dincludes=io.swagger*` | Filtra a árvore exibindo apenas galhos de dependências que contenham o termo pesquisado. |
| `mvn clean package -U` | Força a atualização imediata de todas as dependências SNAPSHOT ignorando caches locais. |
| `mvn dependency:analyze` | Analisa o código e aponta bibliotecas utilizadas mas não declaradas, ou declaradas e nunca usadas. |
| `mvn release:prepare` | Valida o projeto, roda testes e gera as tags no Git para a release. |
| `mvn release:perform` | Clona a tag Git e sobe os JARs, Sources, Javadocs e assinaturas para o Staging do Maven Central. |

---

## 4. Apêndice — Atualizações & Boas Práticas Modernas

### 4.1 A Nova Era: Sonatype Central Portal (2024+)
O processo tradicional demonstrado no curso (abrir tickets no Jira da Sonatype, configurar `oss.sonatype.org` e usar o Nexus Staging Plugin) foi reformulado:
* **Central Portal (`central.sonatype.com`)**: A Sonatype lançou uma interface unificada moderna.
* **Validação de Namespace por DNS/GitHub**: Não há mais necessidade de intervenção humana em chamados do Jira para registrar novos `groupId`. A posse de domínio ou de conta no GitHub é verificada automaticamente via registros DNS TXT ou repositórios temporários.
* **Novo Plugin Oficial**:
  ```xml
  <plugin>
      <groupId>org.sonatype.central</groupId>
      <artifactId>central-publishing-maven-plugin</artifactId>
      <version>0.4.0</version>
      <extensions>true</extensions>
      <configuration>
          <publishingServerId>central</publishingServerId>
          <autoPublish>true</autoPublish>
      </configuration>
  </plugin>
  ```
  Permite publicar diretamente no Maven Central com validação imediata e publicação atômica automatizada.

### 4.2 Prevenção Ativa com o Maven Enforcer Plugin
Para impedir que colisões como a do `NoSuchMethodError` passem despercebidas até a produção, a boa prática moderna é incluir o **`maven-enforcer-plugin`** no pipeline:
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-enforcer-plugin</artifactId>
    <version>3.4.1</version>
    <executions>
        <execution>
            <id>enforce-dependency-convergence</id>
            <goals>
                <goal>enforce</goal>
            </goals>
            <configuration>
                <rules>
                    <!-- Falha o build se houver divergência de versões transitivas no classpath -->
                    <dependencyConvergence/>
                    <!-- Bloqueia dependências indesejadas ou legadas -->
                    <bannedDependencies>
                        <excludes>
                            <exclude>commons-logging:commons-logging</exclude>
                            <exclude>log4j:log4j</exclude>
                        </excludes>
                    </bannedDependencies>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```
* A regra `<dependencyConvergence/>` força o desenvolvedor a resolver explicitamente qualquer ambiguidade de versão transitiva antes que o código chegue aos testes.
