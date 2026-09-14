# Questões Teóricas - Capítulo 01: Spring Boot 4 Quick Start

### Questão 1
O que é o Spring Boot e qual é o seu principal objetivo em comparação ao ecossistema Spring tradicional?

> [!faq]- Resposta
> O Spring Boot é um framework construído sobre a plataforma Spring que visa simplificar a criação e execução de aplicações Java prontas para produção. Seu principal objetivo é eliminar a necessidade de configurações boilerplate complexas (como arquivos XML extensos) através do conceito de **Convenção sobre Configuração**, fornecendo servidores web embutidos (como o Tomcat) e gerenciando dependências automaticamente através dos Starters.

---

### Questão 2
Qual é o papel da anotação `@SpringBootApplication` e quais são as três anotações internas que ela combina?

> [!faq]- Resposta
> A anotação `@SpringBootApplication` é a anotação principal colocada na classe `main`. Ela combina:
> 1. `@SpringBootConfiguration`: Declara a classe como uma fonte de configuração de beans Spring.
> 2. `@EnableAutoConfiguration`: Habilita a autoconfiguração do Spring Boot com base nas dependências presentes no classpath.
> 3. `@ComponentScan`: Ativa a varredura automática de componentes (`@Component`, `@Service`, `@Repository`, `@RestController`) no pacote da classe anotada e em seus subpacotes.

---

### Questão 3
Como o Spring Boot gerencia o ciclo de vida e a compatibilidade de versões de dependências através dos Starters do Maven?

> [!faq]- Resposta
> O Spring Boot utiliza o recurso de **BOM (Bill of Materials)** através do `spring-boot-starter-parent`. Ao declarar dependências "starter" (como `spring-boot-starter-web`), o desenvolvedor não precisa especificar a versão da biblioteca. O Spring Boot gerencia e garante a compatibilidade testada de centenas de bibliotecas de terceiros automaticamente.

---

### Questão 4
O que faz a ferramenta `spring-boot-devtools` e qual o seu comportamento durante o desenvolvimento?

> [!faq]- Resposta
> O Spring Boot DevTools oferece recursos adicionais para melhorar a experiência do desenvolvedor em tempo de criação do código. Suas principais funcionalidades são:
> * **Automatic Restart**: Reinicia a aplicação rapidamente sempre que arquivos do classpath são alterados.
> * **LiveReload**: Atualiza automaticamente o navegador quando arquivos estáticos ou templates HTML mudam.
> * Desativação automática de caches de templates (como Thymeleaf) durante o desenvolvimento.

---

### Questão 5
Para que serve o **Spring Boot Actuator** e quais são seus principais endpoints nativos de monitoramento?

> [!faq]- Resposta
> O Spring Boot Actuator fornece recursos prontos para produção que permitem monitorar e gerenciar a aplicação. Seus principais endpoints incluem:
> * `/actuator/health`: Exibe o estado de saúde da aplicação (e conexões como Banco de Dados).
> * `/actuator/info`: Exibe informações personalizadas sobre a aplicação.
> * `/actuator/metrics`: Exibe métricas de desempenho (uso de memória, CPU, sessões).
> * `/actuator/env`: Exibe as propriedades do ambiente configuradas.

---

### Questão 6
Como são gerenciadas as propriedades de configuração no Spring Boot através do arquivo `application.properties` ou `application.yml`?

> [!faq]- Resposta
> O Spring Boot lê automaticamente esses arquivos localizados na pasta `src/main/resources`. As propriedades podem alterar portas de servidor (ex: `server.port=8081`), conexões com bancos de dados (`spring.datasource.url`), níveis de log (`logging.level.root`) ou definir valores customizados injetados no código com a anotação `@Value("${minha.propriedade}")`.

---

### Questão 7
Qual é a diferença entre um servidor web embutido (*Embedded Server*) e um servidor tradicional implantado via arquivo WAR?

> [!faq]- Resposta
> Um servidor embutido (como o Tomcat ou Jetty nativo do Spring Boot) é empacotado diretamente dentro do arquivo `.jar` executável da aplicação, permitindo que ela rode de forma autônoma com o comando `java -jar app.jar`. Já no modelo tradicional com WAR, a aplicação precisa ser compilada e implantada dentro de um servidor web/aplicação pré-instalado externamente.

---

### Questão 8
O que é o repositório **Spring Initializr** (`start.spring.io`) e como ele auxilia no início de um projeto?

> [!faq]- Resposta
> O Spring Initializr é um gerador de projetos baseado em ambiente web ou IDE que permite estruturar a arquitetura inicial da aplicação. Nele, o desenvolvedor escolhe a ferramenta de build (Maven/Gradle), versão da linguagem Java, versão do Spring Boot e insere as dependências desejadas, gerando um projeto pré-configurado pronto para download.

---

### Questão 9
Como funciona a prioridade de substituição de propriedades de configuração em ambiente de produção no Spring Boot?

> [!faq]- Resposta
> O Spring Boot utiliza uma ordem de precedência bem definida para sobrescrever configurações. Argumentos de linha de comando (`--server.port=8082`) e Variáveis de Ambiente do Sistema Operacional têm prioridade sobre as propriedades declaradas dentro do arquivo `application.properties` empacotado no JAR.

---

### Questão 10
Por que o pacote raiz onde a classe `@SpringBootApplication` reside é fundamental para o correto funcionamento da injeção de dependências no Spring Boot?

> [!faq]- Resposta
> Porque o `@ComponentScan` embutido no `@SpringBootApplication` escaneia por padrão a classe principal e todos os pacotes filhos (subpacotes). Se uma classe anotada com `@Component` ou `@Service` for criada num pacote fora do escopo ou paralelo ao pacote da classe principal, o Spring não a encontrará e falhará ao injetar a dependência (`NoSuchBeanDefinitionException`).
