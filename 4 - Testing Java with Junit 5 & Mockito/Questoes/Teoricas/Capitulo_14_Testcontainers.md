# Questões Teóricas - Capítulo 14: Testes de Integração com Testcontainers

Testes de fixação sobre Testcontainers, integração com Docker, gerenciamento dinâmico de portas, `@ServiceConnection`, sincronização e bancos de dados reais.

---

### 1. O que é a biblioteca Testcontainers e qual problema crítico em relação aos bancos em memória (como H2) ela foi projetada para resolver?

> [!faq]- 👀 Ver Resposta
> O Testcontainers é uma biblioteca Java que orquestra programaticamente contêineres Docker leves e descartáveis durante a execução de testes. Ela resolve a "falsa sensação de segurança" gerada por bancos em memória como o H2, cuja sintaxe SQL, tipos de dados, dialetos, funções embutidas e comportamentos de concorrência divergem frequentemente dos bancos de dados reais usados em produção (como MySQL, PostgreSQL ou Oracle). Com o Testcontainers, os testes executam contra instâncias reais e idênticas às de produção.

---

### 2. Como o Docker e o Testcontainers gerenciam as portas de rede dos serviços instanciados para evitar conflitos de porta na máquina hospedeira?

> [!faq]- 👀 Ver Resposta
> O Testcontainers instrui o daemon do Docker a expor as portas internas padrão do serviço (como a porta `3306` do MySQL ou `5432` do PostgreSQL) em **portas TCP livres aleatórias e efêmeras** sorteadas pelo sistema operacional no host. Isso evita conflitos com bancos de dados reais que o desenvolvedor já tenha rodando localmente na máquina e permite que múltiplas suítes de testes executem concorrentemente sem colisão de portas.

---

### 3. Quais são as três dependências principais no `pom.xml` necessárias para habilitar o Testcontainers com MySQL no Spring Boot 3+?

> [!faq]- 👀 Ver Resposta
> 1. **`spring-boot-testcontainers`:** Módulo oficial do Spring Boot que provê suporte nativo e a funcionalidade de conexão automática via anotações.
> 2. **`org.testcontainers:junit-jupiter`:** Integração oficial entre o ciclo de vida do Testcontainers e a engine do JUnit 5.
> 3. **`org.testcontainers:mysql`:** O módulo especializado que encapsula a imagem oficial do MySQL com configurações prontas de inicialização e health check.

---

### 4. Qual é o papel da anotação `@ServiceConnection` introduzida no Spring Boot 3 e qual configuração manual obsoleta ela substituiu?

> [!faq]- 👀 Ver Resposta
> A anotação `@ServiceConnection` detecta automaticamente a instância do contêiner instanciado (como `MySQLContainer`) e injeta dinamicamente as credenciais e a URL JDBC correta (com a porta aleatória sorteada pelo Docker) nas propriedades de DataSource do Spring Boot (`spring.datasource.url`, `username`, `password`). Ela substituiu a necessidade de declarar blocos manuais e verbosos da anotação legada `@DynamicPropertySource`.

---

### 5. O que fazem, respectivamente, as anotações `@Testcontainers` e `@Container` em uma classe de testes?

> [!faq]- 👀 Ver Resposta
> * **`@Testcontainers`:** Anotação a nível de classe que ativa a extensão do JUnit 5 responsável por rastrear os contêineres declarados e gerenciar seus ciclos de vida globais durante a execução da suíte.
> * **`@Container`:** Marca um campo estático ou de instância de um contêiner (como `MySQLContainer`), instruindo o JUnit a inicializá-lo automaticamente antes do início dos testes e encerrá-lo/destruí-lo ao término da suíte.

---

### 6. Por que é considerada uma boa prática especificar uma tag com a versão exata da imagem Docker (ex: `mysql:8.4.0`) em vez de utilizar a tag genérica `latest`?

> [!faq]- 👀 Ver Resposta
> Porque a tag `latest` é volátil e mutável: a qualquer momento os mantenedores da imagem podem atualizar a versão subjacente do banco de dados, introduzindo incompatibilidades, quebras de sintaxe SQL ou mudanças de drivers que façam a suíte de testes quebrar subitamente sem que nenhuma linha de código da aplicação tenha sido modificada. Usar uma versão cravada (como `8.4.0`) garante determinismo, reprodutibilidade e fidelidade estrita à versão instalada nos servidores de produção.

---

### 7. O que é o contêiner utilitário `Ryuk` iniciado automaticamente pelo Testcontainers em segundo plano?

> [!faq]- 👀 Ver Resposta
> O `Ryuk` (ou *Resource Reaper*) é um contêiner auxiliar leve gerenciado pelo Testcontainers que monitora a execução do processo Java. Sua função primordial é garantir a limpeza do ambiente de testes: caso a execução dos testes na JVM seja abortada abruptamente, caia por falta de memória ou o desenvolvedor encerre a IDE à força, o `Ryuk` detecta a morte do processo e deleta imediatamente todos os volumes e contêineres órfãos criados pela suíte, evitando que recursos de disco fiquem vazando na máquina.

---

### 8. Qual é a causa do erro de concorrência *"Mapped port can only be obtained when the container is running"* em suítes com `@TestInstance(PER_CLASS)`?

> [!faq]- 👀 Ver Resposta
> Esse erro ocorre por uma condição de corrida (*Race Condition*): o Spring Boot tenta inicializar seu `ApplicationContext` e resolver as propriedades de conexão com o banco de dados antes que o contêiner Docker tenha finalizado totalmente sua rotina interna de boot e configurado o mapeamento das portas aleatórias de rede, fazendo com que a requisição de consulta à porta retorne uma falha de contêiner não iniciado.

---

### 9. Como a inicialização manual do contêiner dentro de um bloco estático (`static { container.start(); }`) soluciona esse problema de sincronização?

> [!faq]- 👀 Ver Resposta
> Em Java, blocos estáticos (`static { }`) são executados pela Máquina Virtual no momento exato em que a classe de teste é carregada na memória pelo ClassLoader, **antes** de qualquer anotação ou rotina de injeção de dependências do Spring Boot ser inicializada. Ao chamar `container.start()` dentro desse bloco, garantimos que a thread principal do Java ficará bloqueada aguardando o contêiner subir por completo e estar pronto para receber conexões antes de liberar o Spring para configurar seu `DataSource`.

---

### 10. Além de bancos relacionais tradicionais, quais outros tipos de serviços e ecossistemas de infraestrutura o Testcontainers é capaz de instanciar para testes de integração?

> [!faq]- 👀 Ver Resposta
> O Testcontainers possui módulos especializados capazes de orquestrar praticamente qualquer tecnologia empacotada em imagem Docker, incluindo bancos NoSQL (como MongoDB, Redis, Cassandra), corretores de mensageria e streaming de eventos (como Apache Kafka, RabbitMQ), plataformas de busca (Elasticsearch, OpenSearch) e até emuladores de serviços de computação em nuvem (como LocalStack para simular a AWS).
