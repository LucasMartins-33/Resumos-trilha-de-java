# Questões Teóricas - Capítulo 03: Adicionando JUnit 5 a um Projeto Gradle

Testes de fixação sobre ferramentas de build com Gradle, configuração do JUnit Platform e uso do Gradle Wrapper.

---

### 1. Qual é o papel do arquivo `build.gradle` em um projeto Java e como ele se compara ao `pom.xml` do Maven?

<details>
<summary>👀 Ver Resposta</summary>

O `build.gradle` é o script central de configuração do Gradle, responsável por definir plugins, repositórios, dependências e tarefas customizadas (tasks) do projeto. Enquanto o `pom.xml` do Maven utiliza XML declarativo estático, o `build.gradle` utiliza uma DSL (Domain-Specific Language) baseada em linguagens dinâmicas como Groovy ou Kotlin, permitindo scripts mais concisos e programáveis para pipelines de compilação complexas.
</details>

---

### 2. O que representa a configuração `testImplementation` no bloco `dependencies` do Gradle e por que ela deve ser usada em vez de `implementation` para o JUnit 5?

<details>
<summary>👀 Ver Resposta</summary>

A diretiva `testImplementation` restringe o escopo da dependência estritamente à compilação e execução do código de teste (`src/test/java`), não expondo essa biblioteca para o código de produção (`src/main/java`) nem empacotando-a no artefato final. Se fosse utilizada a diretiva `implementation`, o JUnit 5 estaria acessível dentro do código de produção e seria enviado desnecessariamente para o ambiente de produção.
</details>

---

### 3. Por que a diretiva `useJUnitPlatform()` dentro do bloco `test { ... }` no `build.gradle` é obrigatória para a execução dos testes com JUnit 5?

<details>
<summary>👀 Ver Resposta</summary>

Historicamente, o executor de testes nativo do Gradle foi desenhado para o JUnit 4 ou TestNG. A instrução `useJUnitPlatform()` informa explicitamente à task `test` do Gradle que ela deve utilizar o motor da JUnit Platform para descobrir e executar os testes. Sem essa instrução, o Gradle tentará procurar testes usando a engine do JUnit 4 e ignorará completamente as classes e anotações do JUnit Jupiter (`org.junit.jupiter.api.*`).
</details>

---

### 4. O que acontece na prática se você declarar a dependência `testImplementation 'org.junit.jupiter:junit-jupiter:...'`, criar testes com `@Test` do Jupiter, mas esquecer de configurar `useJUnitPlatform()` no `build.gradle`?

<details>
<summary>👀 Ver Resposta</summary>

O código de teste compilará sem erros, mas ao executar a task de teste via terminal (`./gradlew test`), o Gradle relatará que nenhum teste foi encontrado ("0 tests executed" ou "UP-TO-DATE") ou considerará a build aprovada sem de fato rodar seus testes unitários, criando uma falsa sensação de estabilidade.
</details>

---

### 5. Qual é o objetivo da configuração `testLogging { showStandardStreams = true }` dentro da task de testes do Gradle?

<details>
<summary>👀 Ver Resposta</summary>

Por padrão, o Gradle silencia as saídas padrão do Java (`System.out` e `System.err`) geradas durante a execução dos métodos de teste para manter os logs do terminal limpos. A propriedade `showStandardStreams = true` instrui o Gradle a encaminhar essas impressões diretamente para o console do terminal durante o build, o que auxilia no diagnóstico imediato e na depuração de valores intermediários.
</details>

---

### 6. O que é o Gradle Wrapper (arquivos `gradlew` e `gradlew.bat`) e qual a principal vantagem de utilizá-lo em uma equipe de desenvolvimento?

<details>
<summary>👀 Ver Resposta</summary>

O Gradle Wrapper é um script utilitário acompanhado de uma pasta `.gradle/wrapper` que empacota e baixa automaticamente a versão exata do Gradle especificada para o projeto, caso ela ainda não esteja instalada no computador. A principal vantagem é garantir total consistência e reprodutibilidade de build: todos os desenvolvedores e os servidores de Integração Contínua (CI) executarão o projeto sob a mesma versão idêntica do Gradle, sem conflitos com instalações globais do sistema operacional.
</details>

---

### 7. No Linux ou macOS, por que utilizamos a sintaxe `./gradlew` (com ponto e barra) em vez de apenas digitar `gradlew`?

<details>
<summary>👀 Ver Resposta</summary>

Em sistemas baseados em Unix (Linux e macOS), por razões de segurança, o diretório atual de trabalho (`.`) não faz parte da variável de ambiente `$PATH`. O prefixo `./` instrui explicitamente o shell a executar o script executável localizado exatamente no diretório corrente onde o comando foi emitido.
</details>

---

### 8. Qual é a finalidade de encadear a task `clean` antes da task `test` (ex: `./gradlew clean test`)?

<details>
<summary>👀 Ver Resposta</summary>

O Gradle possui um mecanismo avançado de cache incremental (Up-to-Date Checks). Se o código-fonte e os testes não foram modificados desde o último build, o Gradle pode reaproveitar o resultado anterior e não reexecutar os testes. O comando `clean` deleta completamente o diretório de compilação (`build/`), garantindo que o projeto seja recompilado do zero e que toda a suíte de testes seja executada de fato, evitando relatórios desatualizados causados por cache.
</details>

---

### 9. Onde o Gradle salva os relatórios detalhados de execução de testes em HTML e como eles podem ser consultados?

<details>
<summary>👀 Ver Resposta</summary>

Ao término da execução da task `test`, o Gradle gera automaticamente um relatório HTML interativo localizado no caminho `build/reports/tests/test/index.html`. Esse arquivo pode ser aberto em qualquer navegador web e apresenta métricas completas: total de testes executados, quantidade de sucessos, falhas, tempo de execução por classe e a rastreabilidade de stack traces de eventuais falhas.
</details>

---

### 10. Por que é necessário realizar a sincronização ("Reload/Sync Gradle Changes") na IDE após editar dependências no arquivo `build.gradle`?

<details>
<summary>👀 Ver Resposta</summary>

A edição de texto no arquivo `build.gradle` não atualiza o modelo de memória da IDE em tempo real. A sincronização força a IDE a executar o daemon do Gradle em segundo plano para resolver o grafo de dependências, baixar os binários (.jar) para a pasta de cache local do Gradle e reconfigurar as bibliotecas externas no classpath do projeto, permitindo que a IDE reconheça as classes e forneça autocompletion e validações sintáticas.
</details>
