# Questões Teóricas - Capítulo 04 (Getting Started with Maven)

**1. O que é o POM:** O que significa a sigla POM no Apache Maven e qual é o seu papel fundamental na arquitetura de um projeto?
> [!faq]- 👀 Ver Resposta
> POM significa **Project Object Model**. É o arquivo de configuração declarativo em XML (`pom.xml`) que atua como o coração de todo projeto Maven. Ele contém as coordenadas únicas do projeto, metadados, dependências externas, plugins, metas de compilação e perfis de execução, servindo como a fonte única da verdade para o ciclo de vida do build.

**2. Coordenadas GAV:** Explique o que são as coordenadas Maven (frequentemente chamadas de coordenadas GAV). O que cada um dos três elementos representa e qual o papel do quarto elemento (`packaging`)?
> [!faq]- 👀 Ver Resposta
> As coordenadas GAV identificam unicamente qualquer artefato no ecossistema Maven:
> * `groupId`: Identifica o domínio, organização ou empresa responsável (geralmente o domínio invertido, ex: `com.empresa.projeto`).
> * `artifactId`: O nome do projeto, módulo ou biblioteca específica (ex: `financeiro-core`).
> * `version`: O número de versão daquele artefato (ex: `1.0.0-SNAPSHOT` ou `2.3.1`).
> O elemento opcional `packaging` define o tipo de pacote que o build deve produzir (ex: `jar`, `war`, `pom` ou `ear`), sendo `jar` o padrão caso seja omitido.

**3. Standard Directory Layout:** Por que o Maven adota o princípio de *"Convenção sobre Configuração"* (*Convention over Configuration*)? Cite os caminhos padrão para código-fonte Java principal, recursos e testes unitários.
> [!faq]- 👀 Ver Resposta
> Adota esse princípio para eliminar a necessidade de configurar manualmente caminhos de diretórios em todo projeto novo. Qualquer desenvolvedor ou ferramenta reconhece instantaneamente a estrutura de qualquer projeto Maven no mundo. Os caminhos padrão são:
> * `src/main/java`: Classes-fonte da aplicação Java.
> * `src/main/resources`: Arquivos de configuração, propriedades, XMLs e recursos estáticos empacotados no JAR.
> * `src/test/java`: Classes de teste unitário/integrado (não incluídas no JAR final).
> * `src/test/resources`: Recursos e configurações exclusivos para execução da suíte de testes.

**4. A Pasta `target/`:** O que é a pasta `target/` gerada na raiz de um projeto Maven e por que ela deve constar obrigatoriamente no arquivo `.gitignore`?
> [!faq]- 👀 Ver Resposta
> A pasta `target/` é o diretório de saída transitório onde o Maven deposita todo o produto das operações de build: classes compiladas (`target/classes`), relatórios de testes e o arquivo de pacote final (`.jar` ou `.war`). Ela deve constar no `.gitignore` porque contém arquivos binários derivados, gerados automaticamente e voláteis, que poluiriam o histórico do Git e aumentariam o tamanho do repositório desnecessariamente.

**5. O Ciclo de Vida: `clean` e `package`:** Descreva o que acontece nos bastidores quando um desenvolvedor executa o comando `mvn clean package`.
> [!faq]- 👀 Ver Resposta
> O comando invoca duas fases distintas do Maven:
> 1. `clean`: Aciona o `maven-clean-plugin`, que apaga completamente o diretório `target/` da máquina local, eliminando classes compiladas antigas e garantindo um build limpo.
> 2. `package`: Aciona o ciclo de vida padrão, executando sequencialmente as fases anteriores: `validate` (valida a integridade do POM), `compile` (compila os fontes em `target/classes`), `test` (executa os testes unitários via Surefire) e, finalmente, `package` (empacota as classes e recursos no arquivo `.jar` dentro de `target/`).

**6. Resolução Automática de Dependências:** O que o Maven faz quando encontra uma nova tag `<dependency>` declarada dentro do `pom.xml` ao executar um build?
> [!faq]- 👀 Ver Resposta
> O Maven verifica primeiro se o artefato especificado por aquelas coordenadas GAV já existe no repositório de cache local da máquina (`~/.m2/repository`). Se não existir, conecta-se ao repositório remoto configurado (por padrão o **Maven Central**), realiza o download do arquivo `.jar`, de seu respectivo arquivo `.pom` e de todas as suas **dependências transitivas**, armazena-os no cache local e os injeta automaticamente no Classpath de compilação/execução.

**7. Versões SNAPSHOT vs Versões Estáveis:** Qual é o significado do sufixo `-SNAPSHOT` em uma versão Maven (ex: `1.0-SNAPSHOT`) em termos de mutabilidade e propósito de desenvolvimento?
> [!faq]- 👀 Ver Resposta
> O sufixo `-SNAPSHOT` sinaliza que o artefato é uma versão de desenvolvimento em progresso (*work-in-progress*) e, portanto, **mutável**. Diferente de releases estáveis (que são imutáveis e fixas), o Maven entende que um SNAPSHOT pode ser atualizado a qualquer momento no repositório por outros desenvolvedores, checando periodicamente se há novas compilações mais recentes do mesmo código.

**8. Configuração do Compilador Java no POM:** Antigamente utilizava-se `<maven.compiler.source>` e `<target>`. Qual propriedade moderna introduzida a partir do Java 9 é recomendada para definir a versão da linguagem e compatibilidade de API no Maven?
> [!faq]- 👀 Ver Resposta
> A propriedade moderna recomendada é `<maven.compiler.release>` (ou o argumento `--release` do `javac`). Diferente de `source`/`target` (que apenas alteravam a sintaxe e o formato do bytecode, mas podiam acidentalmente utilizar APIs da JVM mais nova instalada), o parâmetro `release` garante que tanto a sintaxe quanto as APIs da plataforma padrão do Java sejam estritamente limitadas à versão do JDK configurada (ex: `<maven.compiler.release>17</maven.compiler.release>`).

**9. O Maven Wrapper (`mvnw`):** O que é o Maven Wrapper (`mvnw` / `mvnw.cmd`), por que ele é fornecido por padrão em inicializadores modernos como o Spring Initializr e qual problema de ambiente ele resolve?
> [!faq]- 👀 Ver Resposta
> O Maven Wrapper é um script utilitário (`mvnw` para Unix e `mvnw.cmd` para Windows) acompanhado de um arquivo JAR leve sob a pasta `.mvn/wrapper/`. Ele resolve o problema de incompatibilidade de ferramentas entre membros da equipe e esteiras de CI/CD: ao executar `./mvnw clean package`, o script verifica se o Maven está instalado na versão exata especificada nas configurações; se não estiver, faz o download automático daquela versão do Maven em segundo plano, dispensando qualquer instalação manual prévia no sistema operacional.

**10. Integração com IDEs (IntelliJ IDEA):** Como o IntelliJ IDEA ou Eclipse utilizam o `pom.xml` para configurar o projeto? É necessário commitar arquivos específicos de projeto da IDE (`.iml`, pasta `.idea/`) no repositório Git?
> [!faq]- 👀 Ver Resposta
> As IDEs modernas tratam o `pom.xml` como a fonte primária de verdade: lêem o arquivo para identificar quais diretórios são fontes ou testes, baixam as dependências para autocompletion, configuram a versão do SDK e mapeiam as bibliotecas no Classpath do editor. Não é necessário commitar arquivos da IDE (`.idea/`, `.iml`) no Git, pois qualquer desenvolvedor pode simplesmente clonar o repositório puro e importar o projeto diretamente a partir do `pom.xml`.
