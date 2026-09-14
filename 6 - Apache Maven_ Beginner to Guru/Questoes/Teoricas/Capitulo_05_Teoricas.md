# Questões Teóricas - Capítulo 05 (Maven Basics)

**1. A Estrutura de Coordenadas e Repositórios:** Como o Maven traduz as coordenadas GAV (`groupId`, `artifactId`, `version`) na estrutura física de pastas dentro do repositório local `~/.m2/repository/`? Dê um exemplo prático.
> [!faq]- 👀 Ver Resposta
> O Maven converte cada ponto (`.`) do `groupId` em uma barra de subdiretório, seguida pelo `artifactId`, pela `version` e pelo arquivo JAR nomeado. Por exemplo, a dependência:
> * `groupId`: `org.springframework.boot`
> * `artifactId`: `spring-boot-starter-web`
> * `version`: `3.2.0`
> Será armazenada no disco exatamente no caminho:
> `~/.m2/repository/org/springframework/boot/spring-boot-starter-web/3.2.0/spring-boot-starter-web-3.2.0.jar`.

**2. O Super POM:** O que é o Super POM no Maven? Se o seu `pom.xml` não tiver nenhuma configuração de plugins ou repositórios, de onde vêm as configurações padrão que permitem ao Maven compilar seu projeto?
> [!faq]- 👀 Ver Resposta
> O Super POM é o arquivo POM mestre embutido no núcleo da instalação do Apache Maven. Ele atua de forma idêntica à classe `java.lang.Object` no Java: **todo** `pom.xml` herda implicitamente do Super POM. É nele que estão definidas as convenções universais do Maven: estrutura de pastas padrão (`src/main/java`), repositório remoto padrão (**Maven Central**) e o mapeamento dos plugins essenciais de compilação, teste e empacotamento.

**3. O Effective POM:** O que é o *Effective POM* e qual comando da CLI do Maven (`mvn ...`) permite inspecionar esse modelo em tempo de execução?
> [!faq]- 👀 Ver Resposta
> O Effective POM é o modelo de projeto final e consolidado resultante da mesclagem hierárquica completa de: Super POM + configurações globais/usuário (`settings.xml`) + POMs pais herdados + o `pom.xml` específico do projeto + perfis ativos. Ele pode ser inspecionado executando o comando `mvn help:effective-pom`.

**4. Mediação de Dependências ("Nearest Wins"):** Quando duas dependências do seu projeto solicitam versões diferentes de uma mesma biblioteca transitiva (ex: uma pede a versão `1.2` e a outra pede a `2.0`), como o algoritmo *"Nearest Wins"* (*Mais Próximo Vence*) do Maven resolve esse conflito?
> [!faq]- 👀 Ver Resposta
> O algoritmo escolhe a versão da dependência que estiver na menor distância hierárquica (menor profundidade de galhos) em relação à raiz do seu projeto no grafo de dependências:
> * Se uma dependência estiver declarada diretamente no seu `pom.xml` (distância 1) e outra vier como dependência da dependência (distância 2), a declarada diretamente sempre vence.
> * Se ambas estiverem na mesma profundidade na árvore, vence a que tiver sido declarada **primeiro** no seu `pom.xml` (*First Declaration Wins*).

**5. O Bloco `<dependencyManagement>`:** Qual é a função estratégica da tag `<dependencyManagement>` em projetos corporativos ou arquivos POM pai? Declarar uma biblioteca dentro dela força a inclusão do JAR no projeto filho?
> [!faq]- 👀 Ver Resposta
> A tag `<dependencyManagement>` atua exclusivamente como uma tabela centralizadora de padronização de versões e regras de exclusão. **Ela não inclui o JAR no projeto nem força seu download**. O projeto filho só receberá o JAR se declarar a dependência explicitamente no seu bloco `<dependencies>`, com o benefício de poder omitir a tag `<version>`, herdando com segurança a versão homologada no `<dependencyManagement>`.

**6. Escopos de Dependência (Scopes):** Explique detalhadamente o que significam os escopos `compile`, `provided`, `runtime` e `test` no Maven, indicando em quais fases (compilação, teste, empacotamento) cada um atua.
> [!faq]- 👀 Ver Resposta
> * `compile` (padrão): Presente em todas as etapas (compilação, execução de testes e empacotamento final no JAR/WAR).
> * `provided`: Presente na compilação e execução de testes, mas **não é empacotado** no artefato final, pois presume-se que o ambiente de execução (ex: servidor Tomcat ou o JDK) fornecerá a biblioteca em tempo de execução (ex: Servlet API).
> * `runtime`: Não é necessário na compilação do código Java, mas é indispensável para os testes e para o empacotamento final em execução (ex: drivers JDBC de banco de dados).
> * `test`: Utilizado estritamente para compilar e rodar a suíte de testes unitários/integrados. Não fica disponível para as classes principais em `src/main` e não é incluído no JAR de produção (ex: JUnit, Mockito).

**7. Escopo `import`:** Para que serve o escopo especial `<scope>import</scope>` e com qual elemento ele deve ser obrigatoriamente utilizado dentro de `<dependencyManagement>`?
> [!faq]- 👀 Ver Resposta
> O escopo `import` é utilizado exclusivamente dentro da tag `<dependencyManagement>` em dependências com `<type>pom</type>`. Ele permite **importar e mesclar** um BOM (*Bill of Materials*) externo (como o `spring-boot-dependencies`) sem que o seu projeto precise herdar daquele POM através da tag `<parent>`, permitindo compor múltiplos catálogos de dependências.

**8. Os Três Ciclos de Vida do Maven:** Quais são os 3 ciclos de vida (*Lifecycles*) embutidos e independentes do Maven e qual é o propósito de cada um?
> [!faq]- 👀 Ver Resposta
> 1. **`default` (ou `build`)**: Responsável pelo processo principal de construção da aplicação (validação, compilação, execução de testes, empacotamento, verificação e deploy).
> 2. **`clean`**: Responsável por limpar o projeto, removendo diretórios de builds anteriores (`target/`).
> 3. **`site`**: Responsável por gerar a documentação HTML do projeto, métricas de código e relatórios de conformidade.

**9. Fases do Lifecycle vs Plugins e Goals:** Explique a relação conceitual no Maven entre uma **Fase do Ciclo de Vida** (*Phase*), um **Plugin** e um **Objetivo** (*Goal*).
> [!faq]- 👀 Ver Resposta
> * O Maven em si é apenas um orquestrador vazio guiado por **Fases** sequenciais abstratas (ex: `compile`, `test`, `package`).
> * Quem realmente realiza o trabalho prático são os **Plugins**, que contêm implementações escritas em Java.
> * Cada tarefa executável de um plugin é chamada de **Goal** (ex: `compiler:compile`, `surefire:test`, `jar:jar`).
> Uma Fase é simplesmente um gatilho que executa uma lista ordenada de Goals de plugins vinculados a ela.

**10. Archetypes vs Starters Modernos:** O que são os *Maven Archetypes* (`mvn archetype:generate`) e por que eles foram amplamente substituídos por ferramentas web como o Spring Initializr no desenvolvimento moderno?
> [!faq]- 👀 Ver Resposta
> *Archetypes* são modelos estruturais de projetos (templates) usados para gerar uma árvore de pastas e POM inicial via terminal. No entanto, muitos tornaram-se desatualizados, mantendo versões antigas de plugins e dependências. A comunidade moderna adotou geradores ágeis na nuvem (como o Spring Initializr em `start.spring.io`), que sempre geram projetos configurados com as versões mais recentes, compatibilidade garantida de JDK e dependências de curadoria atualizada.
