# Questões Teóricas - Capítulo 10 (Multi-Module Projects)

**1. O Conceito de Projeto Multimódulo:** Qual é o objetivo arquitetural de dividir uma aplicação corporativa monolítica em um projeto multimódulo no Maven em vez de manter tudo em um único projeto com muitos pacotes?
<details>
<summary>👀 Ver Resposta</summary>

O objetivo é impor **fronteiras de acoplamento físicas e modulares estritas**. Em um único projeto, qualquer classe pode instanciar acidentalmente qualquer outra classe do sistema. Ao isolar o domínio, regras de negócio e infraestrutura em submódulos Maven independentes (ex: `core`, `service`, `web`), define-se formalmente quem pode consumir quem através de dependências explícitas no `pom.xml`, além de viabilizar a reutilização de bibliotecas compartilhadas e builds incrementais otimizados.
</details>

**2. O Papel do Módulo Raiz Aggregator:** O que caracteriza o arquivo `pom.xml` raiz de um projeto agregador e qual tipo de empacotamento (`packaging`) ele deve ter obrigatoriamente?
<details>
<summary>👀 Ver Resposta</summary>

O POM raiz agregador não contém código-fonte Java (`src/` não existe ou é vazia). Sua função é puramente orquestrar o build de múltiplos submódulos através da tag `<modules>`. Ele deve ter obrigatoriamente o tipo de empacotamento configurado como `<packaging>pom</packaging>`.
</details>

**3. Aggregator POM vs Parent POM:** Explique a diferença conceitual sutil entre um **POM Agregador** (*Aggregator*) e um **POM Pai** (*Parent*). Um arquivo `pom.xml` pode desempenhar os dois papéis simultaneamente?
<details>
<summary>👀 Ver Resposta</summary>

* **Agregador**: Conhece os seus filhos (declara a seção `<modules>`). Ele é usado para compilar todos os projetos em lote de cima para baixo.
* **Parent**: É conhecido pelos filhos (os filhos declaram a seção `<parent>`). Ele é usado para fornecer herança de propriedades, dependências e plugins de baixo para cima.
Sim! Na grande maioria dos projetos do mundo real, o POM raiz desempenha **ambos os papéis simultaneamente**, atuando tanto como agregador (`<modules>`) quanto como pai compartilhado (`<parent>`).
</details>

**4. A Tag `<relativePath>`:** O que a tag `<relativePath>` dentro do bloco `<parent>` de um submódulo faz e por que configurá-la como `<relativePath>../pom.xml</relativePath>` (ou deixar o valor padrão) acelera o desenvolvimento local?
<details>
<summary>👀 Ver Resposta</summary>

A tag `<relativePath>` instrui o Maven sobre onde encontrar o arquivo `pom.xml` do projeto pai no sistema de arquivos local antes de tentar buscá-lo no repositório local (`~/.m2`) ou remoto. Ao apontar para a pasta superior (`../pom.xml`), o Maven lê diretamente as alterações feitas no pai em tempo real, sem exigir que o desenvolvedor execute `mvn install` no pai toda vez que alterar uma propriedade ou versão.
</details>

**5. O Mecanismo do Reactor do Maven:** O que é o *Maven Reactor* e como ele calcula a ordem exata de compilação dos módulos em um build multimódulo?
<details>
<summary>👀 Ver Resposta</summary>

O Reactor é o mecanismo interno do Maven encarregado de orquestrar a execução em projetos multimódulos. Ele inspeciona o grafo de dependências entre todos os submódulos declarados e calcula uma **ordem topológica de build**: se o módulo `web-app` depende do módulo `servicos`, e `servicos` depende de `modelo-dados`, o Reactor garante rigorosamente que `modelo-dados` seja compilado primeiro, seguido por `servicos` e, por último, `web-app`.
</details>

**6. A Seção `<dependencyManagement>` em Projetos Multimódulos:** Como a tag `<dependencyManagement>` no POM pai raiz elimina a divergência de versões entre os diferentes submódulos do projeto?
<details>
<summary>👀 Ver Resposta</summary>

No POM pai, declaram-se todas as bibliotecas e suas versões definitivas dentro de `<dependencyManagement>` (inclusive a versão dos próprios submódulos internos, como `${project.version}`). Nos submódulos filhos, os desenvolvedores apenas declaram o `groupId` e `artifactId` das dependências necessárias sem colocar a tag `<version>`. Isso assegura que todos os módulos utilizem exatamente a mesma versão homologada, eliminando riscos de colisão de versões no classpath.
</details>

**7. Compilação Seletiva de Módulos (Flag `-pl`):** Quando você está em um projeto com dezenas de módulos e deseja compilar apenas um módulo específico a partir da raiz, qual argumento de linha de comando você deve utilizar?
<details>
<summary>👀 Ver Resposta</summary>

Utiliza-se a flag `-pl` (*Project List*), passando o nome relativo da pasta ou as coordenadas do módulo. Por exemplo:
```bash
mvn clean compile -pl web-app
```
O Maven ignorará os demais módulos e executará a compilação exclusivamente no módulo especificado.
</details>

**8. Resolução de Dependências do Módulo (Flags `-am` e `-amd`):** O que fazem as flags `-am` (*Also Make*) e `-amd` (*Also Make Dependents*) quando combinadas com a flag `-pl` na linha de comando do Maven?
<details>
<summary>👀 Ver Resposta</summary>

* `-am` (*Also Make*): Compila o projeto selecionado **e todos os projetos upstream dos quais ele depende**. Ex: `mvn compile -pl web-app -am` compilará primeiro as bibliotecas internas que o `web-app` precisa e depois compilará o `web-app`.
* `-amd` (*Also Make Dependents*): Compila o projeto selecionado **e todos os projetos downstream que dependem dele**. Ideal quando você alterou uma classe no módulo `modelo-dados` e deseja recompilar todos os outros serviços que o consomem para checar se algo quebrou.
</details>

**9. Módulos com Tipos de Empacotamento Distintos:** Em um projeto multimódulo típico corporativo, qual é o empacotamento comum de módulos de domínio/bibliotecas em comparação ao módulo final de distribuição?
<details>
<summary>👀 Ver Resposta</summary>

Módulos de domínio, modelo e lógica de negócios compartilhada utilizam empacotamento tradicional `<packaging>jar</packaging>`, gerando bibliotecas comuns de classes. Já o módulo de borda ou entrega final (responsável pela inicialização da aplicação) utiliza empacotamentos executáveis como `<packaging>jar</packaging>` (com Spring Boot repackage/Fat JAR) ou `<packaging>war</packaging>` para implantação em servidores de aplicação web.
</details>

**10. O Argumento `--resume-from` (`-rf`):** O que o argumento `-rf` faz quando um build multimódulo longo com 15 módulos falha no 8º módulo após 10 minutos de execução?
<details>
<summary>👀 Ver Resposta</summary>

O argumento `--resume-from` (ou `-rf :nome-do-modulo`) permite reiniciar o build do Maven exatamente a partir do módulo que falhou, reaproveitando os módulos anteriores que já foram compilados com sucesso na execução passada. Isso economiza um tempo valioso dos desenvolvedores e da esteira de CI/CD ao evitar recompilar desnecessariamente os 7 primeiros módulos que já estavam íntegros.
</details>
