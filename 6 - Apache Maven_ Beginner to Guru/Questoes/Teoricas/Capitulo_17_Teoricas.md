# Questões Teóricas - Capítulo 17 (Maven in the Real World)

**1. O Cenário de Conflito de Classpath ("Classloader Trap"):** O que significa o erro `java.lang.NoSuchMethodError` em tempo de execução e por que ele frequentemente aponta para a existência de múltiplas versões de uma mesma biblioteca no Classpath da aplicação?
<details>
<summary>👀 Ver Resposta</summary>

Significa que a JVM tentou invocar um método em uma classe carregada na memória, mas aquele método específico não existe no bytecode daquela classe. Isso ocorre quando existem duas versões diferentes da mesma biblioteca no Classpath: o Classloader carrega a primeira versão que encontra no disco (frequentemente a versão mais antiga); quando outra parte da aplicação tenta chamar um método introduzido na versão mais nova, o método não está presente na classe ativa na memória, estourando o `NoSuchMethodError`.
</details>

**2. Por que a Mediação "Nearest Wins" Falha quando as Coordenadas GAV Mudam:** Se o Maven possui o algoritmo de mediação *"Nearest Wins"* para resolver versões conflitantes, por que no estudo de caso real do curso tanto a versão antiga (`2.0.0-rc1`) quanto a nova (`2.0.5`) do Swagger Parser foram adicionadas ao Classpath?
<details>
<summary>👀 Ver Resposta</summary>

Porque os mantenedores da biblioteca alteraram as coordenadas Maven entre as versões: a versão antiga usava o `groupId` `io.swagger`, enquanto a versão nova usava `io.swagger.parser.v3`. Como o Maven baseia a mediação de dependências estritamente na correspondência exata de `groupId` e `artifactId`, ele interpretou as duas bibliotecas como se fossem produtos completamente independentes e não conflitantes, incluindo ambos os JARs no Classpath da aplicação.
</details>

**3. Investigação Diagnóstica com `mvn dependency:tree`:** Como a execução do comando `mvn dependency:tree` (especialmente redirecionada para um arquivo de texto) auxilia o engenheiro a isolar a origem de dependências transitivas indesejadas?
<details>
<summary>👀 Ver Resposta</summary>

O comando imprime a hierarquia completa e recortada do grafo de dependências do projeto. Ao pesquisar pelo nome do pacote no arquivo gerado, é possível rastrear visualmente o "caminho de herança" e identificar exatamente qual dependência de primeiro nível (direta) está trazendo a versão legada ou conflitante de forma transitiva, permitindo aplicar exclusões cirúrgicas (`<exclusions>`) ou forçar a resolução no `<dependencyManagement>`.
</details>

**4. Ciclo de Desenvolvimento Local de Bibliotecas com `mvn clean install`:** Quando um desenvolvedor trabalha simultaneamente em uma biblioteca compartilhada e em um microsserviço que a consome na mesma máquina, qual é o papel do comando `mvn clean install` na biblioteca?
<details>
<summary>👀 Ver Resposta</summary>

O `mvn clean install` compila a biblioteca, roda seus testes e copia o JAR resultante (com versão SNAPSHOT) diretamente para a pasta do repositório local da máquina (`~/.m2/repository`). A partir desse instante, o microsserviço consumidor consegue resolver a biblioteca localmente e compilar sem exigir que o desenvolvedor publique a biblioteca em um repositório remoto ou servidor corporativo.
</details>

**5. A Flag `-U` (`--update-snapshots`):** Qual é a utilidade vital do argumento `-U` ao executar builds em projetos que consomem artefatos SNAPSHOT em um ambiente de equipe distribuída?
<details>
<summary>👀 Ver Resposta</summary>

Por padrão, a política de atualização do Maven para versões SNAPSHOT é diária (`daily`). Se um desenvolvedor remoto publicar uma correção crítica de um SNAPSHOT pela manhã no repositório corporativo, o Maven dos demais colegas na equipe não buscará a nova versão até o dia seguinte se já houver uma cópia local baixada. A flag `-U` (ou `--update-snapshots`) obriga o Maven a ignorar o cache de tempo e consultar o repositório remoto imediatamente, baixando a compilação SNAPSHOT mais recente.
</details>

**6. Sobrescrita de Versão em `<dependencyManagement>` vs `<dependencies>`:** No estudo de caso do capítulo, por que tentar sobrescrever a dependência diretamente no bloco `<dependencies>` falhou e foi necessário declará-la dentro de `<dependencyManagement>`?
<details>
<summary>👀 Ver Resposta</summary>

Porque a versão incorreta estava sendo injetada indiretamente através da importação de um BOM (*Bill of Materials*) herdado. A hierarquia de resolução do Maven confere prioridade absoluta à seção `<dependencyManagement>` do POM mais específico (o POM do próprio projeto) sobre qualquer dependência herdada ou transitiva. Declarar a dependência com a versão corrigida dentro de `<dependencyManagement>` força o Maven a alinhar todas as menções àquela biblioteca no projeto.
</details>

**7. Os Quatro Pilares Obrigatórios do Maven Central (Sonatype OSSRH):** Quais são os quatro arquivos complementares e requisitos de qualidade que todo projeto de código aberto deve gerar para ser aceito na publicação do Maven Central?
<details>
<summary>👀 Ver Resposta</summary>

1. **Binário Principal**: O arquivo JAR compilado (`.jar`).
2. **Código-Fonte**: O pacote contendo os arquivos `.java` originais gerado pelo `maven-source-plugin` (`-sources.jar`).
3. **Documentação da API**: O pacote com os Javadocs gerado pelo `maven-javadoc-plugin` (`-javadoc.jar`).
4. **Assinatura Criptográfica PGP/GPG**: Arquivos de assinatura gerados pelo `maven-gpg-plugin` (`.asc`) para comprovar a autoria de cada arquivo.
Além disso, o `pom.xml` deve conter obrigatoriamente metadados completos de licença, desenvolvedores, SCM e URL do projeto.
</details>

**8. O Conceito de Repositório de Staging da Sonatype (`oss.sonatype.org`):** O que acontece durante a fase de Staging quando artefatos são enviados para o repositório OSSRH da Sonatype antes de estarem visíveis publicamente no Maven Central?
<details>
<summary>👀 Ver Resposta</summary>

Os artefatos não entram diretamente no índice público; eles são depositados em um repositório de testes isolado e fechado (*Staging Repository*) exclusivo para a conta do desenvolvedor. A Sonatype executa validações automáticas rigorosas (verificando se todas as assinaturas GPG são válidas, se os hashes conferem e se o Javadoc/Sources estão presentes). Somente após o comando manual ou automatizado de fechamento (*Close*) e liberação (*Release*), os arquivos são promovidos e sincronizados com os espelhos mundiais do Maven Central.
</details>

**9. A Regra `<dependencyConvergence/>` do Maven Enforcer Plugin:** Como a inclusão da regra de convergência de dependências no `maven-enforcer-plugin` protege projetos corporativos contra problemas de Classpath antes que cheguem a produção?
<details>
<summary>👀 Ver Resposta</summary>

A regra `<dependencyConvergence/>` analisa o grafo de dependências completo e **aborta imediatamente o build com erro** se encontrar qualquer divergência de versão transitiva para a mesma biblioteca entre diferentes galhos da árvore. Isso força os engenheiros a definirem explicitamente qual versão deve ser utilizada no `<dependencyManagement>`, eliminando surpresas em tempo de execução causadas pela escolha silenciosa da versão mais próxima pelo Maven.
</details>

**10. A Nova Era do Sonatype Central Portal (2024+):** Quais melhorias o lançamento do **Sonatype Central Portal** trouxe para a publicação de bibliotecas em relação ao processo tradicional legado de abertura de chamados no Jira?
<details>
<summary>👀 Ver Resposta</summary>

O novo Central Portal eliminou completamente o processo manual e lento de solicitação de permissão de `groupId` via tickets de suporte no Jira. Ele oferece uma interface moderna com validação totalmente automatizada de titularidade de domínio (através de registros DNS TXT ou repositórios públicos de verificação no GitHub), além de disponibilizar uma API REST moderna e o novo plugin oficial `central-publishing-maven-plugin`, permitindo validação e publicação atômica direta no CI/CD.
</details>
