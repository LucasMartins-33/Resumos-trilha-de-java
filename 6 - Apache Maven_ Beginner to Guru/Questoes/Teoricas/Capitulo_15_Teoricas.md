# Questões Teóricas - Capítulo 15 (Maven Build Profiles)

**1. O Conceito de Build Profile:** O que é um Maven Build Profile e qual é a diferença conceitual fundamental entre um perfil do Maven e um perfil do Spring Framework?
<details>
<summary>👀 Ver Resposta</summary>

Um Maven Build Profile é um conjunto de diretivas e configurações condicionais no `pom.xml` ou `settings.xml` que altera o comportamento do projeto **em tempo de compilação e empacotamento** (*build time*), podendo modificar propriedades, plugins ou dependências. A diferença fundamental é que perfis do Spring atuam em **tempo de execução** da aplicação (*runtime*), ligando ou desligando componentes (Beans) na memória, enquanto perfis do Maven já encerram sua atuação no momento em que o binário JAR/WAR é produzido.
</details>

**2. Locais de Declaração de Perfis:** Em quais arquivos um perfil do Maven pode ser declarado e qual é o escopo de visibilidade de cada um?
<details>
<summary>👀 Ver Resposta</summary>

* No **`pom.xml` do Projeto**: Escopo local do projeto. É versionado no Git e portátil para qualquer máquina ou esteira de CI/CD.
* No **`~/.m2/settings.xml` (User Settings)**: Escopo do usuário logado na máquina. Disponível para todos os projetos compilados por aquele usuário, ideal para credenciais de servidores e repositórios locais.
* No **`${maven.home}/conf/settings.xml` (Global Settings)**: Escopo de todos os usuários da máquina/servidor (muito raro, usado em servidores corporativos como agentes Jenkins).
* O antigo arquivo `profiles.xml` foi **descontinuado** e não é suportado desde o Maven 3.0.
</details>

**3. Restrições de Elementos em `<profile>`:** Quais elementos comuns do POM são estritamente proibidos dentro da tag `<profile>` no `pom.xml` e por que essa limitação existe?
<details>
<summary>👀 Ver Resposta</summary>

As coordenadas primárias do projeto (`groupId`, `artifactId`, `version`, `packaging`) e a tag de extensões de build (`<build><extensions>`) **não podem** ser declaradas dentro de um `<profile>`. Essa limitação existe porque o Maven precisa determinar a identidade imutável do projeto e inicializar as extensões de protocolo de transporte (como extensões Wagon) antes mesmo de processar e ativar os perfis de compilação.
</details>

**4. O Elemento `<activation>` e Gatilhos Automáticos:** Quais são os quatro principais mecanismos que permitem ao Maven ativar um perfil automaticamente sem intervenção manual na linha de comando?
<details>
<summary>👀 Ver Resposta</summary>

1. `<activeByDefault>`: Ativado automaticamente caso nenhum outro perfil explícito seja passado na linha de comando.
2. `<jdk>`: Ativação baseada na versão do JDK em uso (ex: `<jdk>17</jdk>` ou intervalos como `<jdk>[17,)</jdk>`).
3. `<os>`: Ativação baseada no sistema operacional (família `windows`, `unix`, arquitetura de 64 bits, etc.).
4. `<property>` ou `<file>`: Ativação pela presença de uma propriedade de sistema (ex: `-Denv=prod`) ou pela existência/ausência física de um arquivo específico em disco (`<exists>` ou `<missing>`).
</details>

**5. Ativação e Desativação Manual via CLI (`-P`):** Como um desenvolvedor ativa múltiplos perfis simultaneamente pela linha de comando e qual é a sintaxe exata para desativar um perfil configurado como ativo por padrão?
<details>
<summary>👀 Ver Resposta</summary>

* Para ativar múltiplos perfis: utiliza-se a flag `-P` seguida dos nomes separados por vírgula: `mvn clean package -P perfil1,perfil2`.
* Para desativar um perfil ativo por padrão: prefixa-se o nome do perfil com um ponto de exclamação `!` ou sinal de menos `-`. Ex: `mvn clean package -P \!perfilDefault` (com barra de escape em terminais Linux/macOS Bash) ou `mvn clean package -P -perfilDefault`.
</details>

**6. Riscos de Conflito entre Múltiplos Perfis Ativos:** O que acontece quando dois perfis ativados simultaneamente no mesmo build definem valores divergentes para a mesma propriedade do Maven? O Maven possui uma ordem clara de precedência?
<details>
<summary>👀 Ver Resposta</summary>

O Maven **não possui uma regra formal ou determinística de precedência** para propriedades concorrentes definidas em múltiplos perfis ativos. A resolução pode variar dependendo da ordem interna em que o modelo XML foi parseado ou do sistema operacional, gerando comportamentos imprevisíveis. Por essa razão, a boa prática de engenharia exige que perfis concorrentes sejam mutuamente exclusivos ou que seus identificadores sejam ativados isoladamente.
</details>

**7. Diagnóstico com `mvn help:active-profiles`:** Qual é a função do comando `mvn help:active-profiles` e por que ele é essencial para depuração em ambientes com muitos perfis condicionais?
<details>
<summary>👀 Ver Resposta</summary>

O comando avalia todo o contexto de execução (variáveis de ambiente, JDK atual, sistema operacional, flags de linha de comando e o arquivo `settings.xml`) e imprime no console a lista exata de todos os perfis que o Maven considerou como ativos para aquele build específico, permitindo diagnosticar rapidamente se um perfil deixou de ser ativado devido a um gatilho mal configurado.
</details>

**8. Injeção de Propriedades em Testes (Surefire / Failsafe):** Como perfis do Maven podem ser combinados com o `maven-surefire-plugin` para executar testes de integração contra ambientes dinâmicos (como `localhost`, `QA` ou `UAT`)?
<details>
<summary>👀 Ver Resposta</summary>

Define-se uma propriedade de ambiente no POM (ex: `<TEST_HOST>localhost:8080</TEST_HOST>`) e cria-se perfis que alteram essa propriedade (ex: perfil `qa` define `<TEST_HOST>qa.api.empresa.com</TEST_HOST>`). No plugin do Surefire/Failsafe, injeta-se o valor via tag `<configuration><environmentVariables><TEST_HOST>${TEST_HOST}</TEST_HOST></environmentVariables></configuration>`. O código de teste lê o valor em tempo de execução via `System.getenv("TEST_HOST")`, permitindo alternar o alvo do teste apenas mudando a flag `-P qa`.
</details>

**9. O Anti-Padrão do "Environment Profiling":** No passado, desenvolvedores usavam perfis do Maven para gerar um JAR para `dev`, um JAR para `homolog` e outro JAR para `prod` trocando arquivos `application.properties`. Por que essa prática hoje é considerada um anti-padrão grave?
<details>
<summary>👀 Ver Resposta</summary>

Porque viola o princípio fundamental do desenvolvimento moderno e da metodologia Twelve-Factor App: **"Build once, deploy anywhere"** (Construa uma vez, implante em qualquer lugar). Gerar binários diferentes por ambiente significa que o artefato testado e homologado pela equipe de QA não é exatamente o mesmo binário (bit a bit) que rodará em produção, abrindo margem para bugs silenciosos. A prática recomendada é empacotar um binário 100% idêntico e injetar variáveis e segredos de ambiente em tempo de execução via contêineres/Kubernetes.
</details>

**10. Boas Práticas Modernas para Perfis do Maven:** Em quais cenários legítimos a utilização de perfis do Maven continua sendo uma prática altamente recomendada no ecossistema de engenharia atual?
<details>
<summary>👀 Ver Resposta</summary>

O uso moderno legítimo de perfis foca em **processos de build e governança de CI/CD**:
* **Perfil de Desenvolvimento Local Rápido (`fast-build`)**: Desativa relatórios pesados e linters para acelerar o feedback diário do desenvolvedor.
* **Perfil de Integração Contínua (`ci`)**: Ativa ferramentas de cobertura de código (JaCoCo), linters estritos (Checkstyle/SpotBugs) e análise de vulnerabilidades.
* **Perfil de Lançamento (`release`)**: Ativa geração de Javadocs, Sources e assinatura criptográfica GPG (`maven-gpg-plugin`) exclusivamente no momento de publicar a biblioteca.
</details>
