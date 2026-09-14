# Questões Teóricas - Capítulo 11 (Apache Maven for Spring Boot)

**1. A Cadeia de Herança do `spring-boot-starter-parent`:** Descreva a hierarquia de herança a partir do momento em que um projeto declara o `spring-boot-starter-parent` como seu `<parent>`. Quais arquivos POM estão acima dele?
> [!faq]- 👀 Ver Resposta
> A cadeia de herança completa flui na seguinte ordem:
> `Projeto do Desenvolvedor` ➔ herda de `spring-boot-starter-parent` ➔ herda de `spring-boot-dependencies` (BOM central de versões) ➔ herda de `spring-boot-build` (configurações corporativas de compilação da Spring/Broadcom) ➔ herda do `Super POM` do Apache Maven.

**2. A Mágica dos Starters do Spring Boot:** Por que, ao adicionar uma dependência como `spring-boot-starter-web` ou `spring-boot-starter-data-jpa`, o desenvolvedor não precisa declarar a tag `<version>`?
> [!faq]- 👀 Ver Resposta
> Porque o projeto herda do `spring-boot-dependencies` (através do parent), que contém uma seção `<dependencyManagement>` exaustivamente testada e curada com centenas de bibliotecas compatíveis entre si. O Maven consulta essa tabela herdada e injeta automaticamente a versão exata e estável homologada pelo time do Spring Boot, eliminando conflitos de versões e boilerplate de configuração.

**3. Sobrescrita de Propriedades (*Properties Override*):** Se uma aplicação Spring Boot precisa atualizar a versão do Java ou utilizar uma versão mais recente de uma biblioteca de terceiros gerenciada pelo Spring (como Kafka ou Hibernate), qual é a forma mais limpa de fazer isso no `pom.xml`?
> [!faq]- 👀 Ver Resposta
> Não é necessário redeclarar a dependência completa com a tag `<version>`. O `spring-boot-dependencies` padroniza todas as versões de ferramentas e bibliotecas em propriedades nomeadas. Basta o desenvolvedor declarar a propriedade correspondente dentro da seção `<properties>` do seu próprio `pom.xml` (ex: `<java.version>17</java.version>` ou `<kafka.version>3.6.0</kafka.version>`). A propriedade do POM filho sobrescreve o valor do POM pai por herança.

**4. Anatomia Interna do Fat JAR do Spring Boot:** Descreva a estrutura interna de pastas de um arquivo JAR executável gerado pelo Spring Boot. Onde residem os arquivos `.class` do seu código e onde residem os JARs das dependências externas?
> [!faq]- 👀 Ver Resposta
> O Fat JAR do Spring Boot possui uma estrutura padronizada:
> * `BOOT-INF/classes/`: Contém os arquivos de bytecode compilados do seu código de negócio e os arquivos de recursos (`application.yml`).
> * `BOOT-INF/lib/`: Contém todos os arquivos JAR intactos das dependências e bibliotecas externas.
> * `META-INF/MANIFEST.MF`: Manifesto contendo os ponteiros de inicialização.
> * `org/springframework/boot/loader/`: Classes descompactadas do carregador especial de classes do Spring Boot.

**5. O Papel do `JarLauncher` e do `LaunchedURLClassLoader`:** Por que a JVM pura não consegue rodar um JAR que contém outros JARs aninhados dentro dele e como o `org.springframework.boot.loader.JarLauncher` contorna essa limitação?
> [!faq]- 👀 Ver Resposta
> A especificação padrão da JVM não possui suporte nativo para carregar classes que estão dentro de arquivos `.jar` compactados dentro de outro `.jar` (*nested JARs*). O Spring Boot resolve isso apontando a `Main-Class` do manifesto para o seu próprio carregador, o `JarLauncher`. Quando executado, o `JarLauncher` inicializa um Classloader customizado (`LaunchedURLClassLoader`), que sabe como ler os bytes dos JARs aninhados em `BOOT-INF/lib/` e, em seguida, transfere o controle para a sua classe com o método `main` real (definida no atributo `Start-Class`).

**6. Spring Boot vs `maven-shade-plugin`:** Qual é a vantagem arquitetural da abordagem de JAR aninhado do Spring Boot em relação à abordagem de "achatamento" (*flattening*) tradicional do `maven-shade-plugin`?
> [!faq]- 👀 Ver Resposta
> O Shade Plugin descompacta todos os arquivos `.class` de todas as bibliotecas e os mistura na mesma raiz. Isso frequentemente causa sobreposição acidental de arquivos de configuração com nomes idênticos (como arquivos de licença, metadados de serviços SPI ou propriedades), corrompendo a aplicação. A abordagem do Spring Boot mantém cada dependência como um JAR isolado e intacto dentro de `BOOT-INF/lib/`, preservando a integridade original de cada biblioteca.

**7. Ciclo de Vida: `spring-boot:start` e `spring-boot:stop` em Testes:** Como o `spring-boot-maven-plugin` se integra ao ciclo de vida do Maven para viabilizar testes de integração automatizados ponta a ponta?
> [!faq]- 👀 Ver Resposta
> O plugin fornece os goals `start` e `stop`:
> * O goal `start` é vinculado à fase `pre-integration-test`, inicializando a aplicação Spring Boot em segundo plano na porta configurada.
> * O `maven-failsafe-plugin` executa os testes de integração HTTP contra o servidor ativo na fase `integration-test`.
> * O goal `stop` é vinculado à fase `post-integration-test`, finalizando graciosamente a aplicação e liberando as portas antes que o build seja concluído na fase `verify`.

**8. O "Repackage Trap" em Projetos Multimódulos:** O que acontece se o `spring-boot-maven-plugin` for executado indistintamente em um submódulo de biblioteca compartilhada (ex: `modulo-dominio`) que não possui classe com método `main`? Como corrigir esse problema?
> [!faq]- 👀 Ver Resposta
> O build falha com o erro: `Execution repackage of goal spring-boot-maven-plugin failed: Unable to find main class`. Além disso, se um módulo de biblioteca for empacotado como Fat JAR, outros módulos não conseguirão consumi-lo, pois suas classes ficam isoladas em `BOOT-INF/classes/`. A correção é desativar o repackage nos módulos utilitários declarando a propriedade `<spring-boot.repackage.skip>true</spring-boot.repackage.skip>` ou configurando `<skip>true</skip>` no plugin daquele submódulo.

**9. Metadados para o Spring Boot Actuator (`build-info` e `git.properties`):** Qual é a utilidade do objetivo `build-info` do `spring-boot-maven-plugin` e do plugin `git-commit-id-maven-plugin` para aplicações em produção?
> [!faq]- 👀 Ver Resposta
> Eles geram metadados valiosos sobre o binário compilado:
> * O goal `build-info` cria o arquivo `META-INF/build-info.properties` (com versão, nome do artefato e hora do build).
> * O `git-commit-id-maven-plugin` cria o `git.properties` (com branch, commit hash e autor).
> Esses arquivos são consumidos automaticamente pelo **Spring Boot Actuator** e expostos no endpoint `/actuator/info`, permitindo que equipes de operações e ferramentas de monitoramento auditem exatamente qual versão do código está rodando no contêiner.

**10. Construção de Imagens OCI/Docker com `build-image`:** A partir do Spring Boot 2.3+, o que o comando `mvn spring-boot:build-image` permite realizar sem que o desenvolvedor precise escrever ou manter um arquivo `Dockerfile`?
> [!faq]- 👀 Ver Resposta
> Ele utiliza a especificação **Cloud Native Buildpacks (Paketo)** para inspecionar a aplicação, compilar o código, configurar automaticamente um JRE otimizado e gerar diretamente uma imagem de contêiner OCI compatível com Docker e Kubernetes. O processo organiza o JAR em camadas eficientes (*layered jars*), permitindo que deploys em CI/CD façam cache das dependências imutáveis e enviem apenas a camada leve de código de negócio, acelerando o tempo de publicação.
