# Capítulo 15: Maven Build Profiles

Neste capítulo, exploramos em profundidade os **Perfis de Compilação do Maven (*Maven Build Profiles*)**, um dos recursos mais poderosos e versáteis da ferramenta. Analisamos como parametrizar comportamentos de compilação, alternar dinamicamente destinos de deploy, injetar variáveis em testes automatizados e controlar a ativação automática ou manual de configurações.

---

## 1. Conceitos Fundamentais

### 1.1 O que é um Build Profile?
Um **Build Profile** é um bloco de configurações condicionais que permite modificar ou complementar o modelo de projeto (`pom.xml`) dependendo de parâmetros de execução, ambiente operacional ou argumentos de linha de comando.
* **Flexibilidade**: Permite alterar propriedades (`<properties>`), ativar plugins adicionais, incluir dependências específicas de um ambiente, redefinir destinos de `<distributionManagement>` ou configurar relatórios.
* **Construção Condicional**: Enquanto perfis do Spring atuam em tempo de execução da aplicação (*runtime*), os perfis do Maven atuam durante o **processo de compilação e empacotamento** (*build time*).

---

### 1.2 Onde Declarar os Perfis

```text
1. No pom.xml do Projeto
   └── Portátil: versionado no Git; acessível por toda a equipe e esteiras de CI/CD.

2. No ~/.m2/settings.xml (User Settings)
   └── Específico da máquina/usuário: ideal para credenciais, proxies e servidores internos.

3. No $M2_HOME/conf/settings.xml (Global Settings)
   └── Global da máquina: compartilhado por todos os usuários do servidor (ex: agente Jenkins).

4. profiles.xml
   └── DESCONTINUADO: Não suportado desde o Maven 3.0 (2010).
```

---

### 1.3 Elementos Suportados dentro de um `<profile>`
Diferente da raiz do `pom.xml`, um perfil aceita um conjunto específico de elementos:

| Permitido em `<profile>` | Observações |
| :--- | :--- |
| `<properties>` | Declaração e sobrescrita de propriedades e versões. |
| `<dependencies>` / `<dependencyManagement>` | Adição de bibliotecas sob demanda. |
| `<plugins>` / `<pluginManagement>` | Configuração condicional de plugins de build. |
| `<distributionManagement>` | Redirecionamento dinâmico de repositórios de release/snapshot. |
| `<repositories>` / `<pluginRepositories>` | Apontamento de repositórios adicionais. |
| `<modules>` | Inclusão de submódulos condicionais em projetos multimódulo. |
| `<build>` *(subconjunto estrito)* | Suporta apenas: `<defaultGoal>`, `<resources>`, `<testResources>`, `<finalName>`, `<plugins>`. |

> [!WARNING]
> A tag `<build><extensions>` **não é permitida** dentro de perfis. Extensões de build (como extensões do Maven Wagon) devem residir obrigatoriamente na raiz do `<build>` principal do `pom.xml`.

---

### 1.4 Mecanismos de Ativação de Perfis

Os perfis podem ser ativados de quatro maneiras:

1. **Ativação Padrão (`<activeByDefault>true</activeByDefault>`)**: Ativo automaticamente caso nenhum outro perfil seja solicitado na linha de comando.
2. **Ativação por Ambiente/Sistema**:
   * **Versão do JDK**: Ativado quando compilado em Java 17, 21, etc.
   * **Sistema Operacional**: Ativado em Windows, Linux ou macOS.
   * **Presença de Arquivo**: Ativado caso um arquivo específico exista ou esteja ausente (`<file><exists>...`).
   * **Propriedade de Sistema**: Ativado se uma propriedade `-Denv=qa` estiver presente.
3. **Ativação Manual via CLI**: Ativação explícita com o argumento `-P`.
4. **Ativação Global via `settings.xml`**: Declarado no bloco `<activeProfiles>`.

> [!CAUTION]
> Se múltiplos perfis ativos definirem o mesmo elemento ou propriedade conflitante, o Maven **não possui regra estrita de precedência**; a resolução pode ser não determinística. Evite ativar simultaneamente perfis com valores concorrentes.

---

## 2. Sintaxe, Comandos & Configurações

### 2.1 Declarando Perfis de Publicação no `pom.xml`
Exemplo alternando o destino de deploy entre **Nexus** e **Packagecloud**:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>
    <modelVersion>4.0.0</modelVersion>
    <groupId>guru.springframework</groupId>
    <artifactId>profile-demo</artifactId>
    <version>1.0-SNAPSHOT</version>

    <profiles>
        <!-- Perfil 1: Publicação no Packagecloud -->
        <profile>
            <id>packagecloud</id>
            <activation>
                <activeByDefault>false</activeByDefault>
            </activation>
            <distributionManagement>
                <repository>
                    <id>packagecloud.release</id>
                    <url>packagecloud+https://packagecloud.io/empresa/release</url>
                </repository>
                <snapshotRepository>
                    <id>packagecloud.snapshot</id>
                    <url>packagecloud+https://packagecloud.io/empresa/snapshot</url>
                </snapshotRepository>
            </distributionManagement>
        </profile>

        <!-- Perfil 2: Publicação no Nexus Corporativo Local -->
        <profile>
            <id>nexus_distro</id>
            <distributionManagement>
                <repository>
                    <id>nexus-release</id>
                    <url>http://localhost:8081/repository/nexus-release/</url>
                </repository>
                <snapshotRepository>
                    <id>nexus-snapshot</id>
                    <url>http://localhost:8081/repository/nexus-snapshot/</url>
                </snapshotRepository>
            </distributionManagement>
        </profile>
    </profiles>
</project>
```

---

### 2.2 Injetando Propriedades de Ambiente em Testes (Surefire)
Permite que testes de integração adaptem seu endpoint de destino de acordo com o ambiente selecionado (`localhost`, `QA` ou `UAT`):

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" ...>

    <!-- Propriedade padrão caso nenhum perfil seja ativado -->
    <properties>
        <TEST_HOST>localhost:8080</TEST_HOST>
    </properties>

    <profiles>
        <profile>
            <id>test-qa</id>
            <properties>
                <TEST_HOST>qa.api.exemplo.com</TEST_HOST>
            </properties>
        </profile>
        <profile>
            <id>uat</id>
            <properties>
                <TEST_HOST>uat.api.exemplo.com</TEST_HOST>
            </properties>
        </profile>
    </profiles>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
                <configuration>
                    <!-- Injeta a propriedade do Maven como Variável de Ambiente para a JVM do teste -->
                    <environmentVariables>
                        <TEST_HOST>${TEST_HOST}</TEST_HOST>
                    </environmentVariables>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

No teste em Java:
```java
@Test
void testApiConnection() {
    String host = System.getenv("TEST_HOST");
    System.out.println("Executando teste contra o host: " + host);
    assertNotNull(host);
}
```

---

### 2.3 Ativação Condicional Automática por Sistema / JDK
```xml
<profile>
    <id>jdk21-optimizations</id>
    <activation>
        <jdk>21</jdk>
    </activation>
    <properties>
        <compiler.argument>--enable-preview</compiler.argument>
    </properties>
</profile>

<profile>
    <id>windows-setup</id>
    <activation>
        <os>
            <family>windows</family>
        </os>
    </activation>
    <properties>
        <script.extension>.bat</script.extension>
    </properties>
</profile>
```

---

## 3. Guia de Linha de Comando (CLI)

| Finalidade | Comando Maven |
| :--- | :--- |
| **Inspecionar Perfis Ativos** | `mvn help:active-profiles` |
| **Inspecionar Perfis com Detalhes** | `mvn help:all-profiles` |
| **Ativar um Perfil Específico** | `mvn clean test -P test-qa` |
| **Ativar Múltiplos Perfis** | `mvn clean package -P test-qa,uat` |
| **Desativar Perfil Ativo por Padrão** | `mvn clean package -P \!packagecloud` *(no Linux/Mac bash)* |
| **Desativar e Ativar Simultaneamente** | `mvn clean deploy -P \!packagecloud,nexus_distro` |
| **Desativação Alternativa (com `-`)** | `mvn clean deploy -P -packagecloud,nexus_distro` |

> [!NOTE]
> No terminal Bash/Zsh do Linux e macOS, o ponto de exclamação `!` invoca a expansão do histórico do shell. Por isso, é indispensável usar barra invertida de escape `\!` ou aspas `'-P !perfil'`.

---

## 4. Apêndice — Atualizações & Boas Práticas Modernas

### 4.1 O Princípio "Build Once, Deploy Anywhere" (12-Factor App)
No passado, era costume usar perfis do Maven para compilar múltiplos arquivos JAR/WAR ligeiramente diferentes: um JAR para `dev`, um para `homolog` e outro para `prod` (trocando arquivos `application.properties` embutidos durante o empacotamento).
* **Anti-Padrão Moderno**: Gerar binários diferentes por ambiente quebra a rastreabilidade e a garantia de qualidade (o que foi homologado no teste não é bit a bit o que roda em produção).
* **Boa Prática Atual**: O Maven deve gerar **um único artefato imutável**. Configurações de banco, URLs e senhas de ambiente devem ser fornecidas em tempo de execução via **Variáveis de Ambiente**, **Spring Profiles**, Kubernetes ConfigMaps ou Vault.

### 4.2 Para Que Perfis do Maven Devem Ser Usados Hoje?
No ecossistema corporativo atual, os perfis do Maven continuam indispensáveis, porém com foco em **processos de engenharia e CI/CD**:
1. **Perfil de Desenvolvimento Rápido (`fast` / `dev`)**: Pula geração de relatórios pesados, desativa linters rigorosos e compila incrementalmente para acelerar o feedback loop do desenvolvedor.
2. **Perfil de Integração Contínua (`ci` / `strict`)**: Ativa plugins de cobertura de código (**JaCoCo**), análise estática rigorosa (**SpotBugs**, **Checkstyle**) e verificação de vulnerabilidades de dependências.
3. **Perfil de Release Seguro (`release` / `publish`)**: Ativa o plugin de assinatura criptográfica GPG (`maven-gpg-plugin`) e geração de Javadoc/Sources para envio ao Maven Central.
