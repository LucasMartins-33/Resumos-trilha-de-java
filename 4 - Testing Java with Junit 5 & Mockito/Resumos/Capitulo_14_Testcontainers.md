# Capítulo 14: Testes de Integração com Testcontainers

Neste capítulo subimos de nível nos testes de ponta-a-ponta. O banco em memória H2 é excelente pela agilidade, mas ele difere dos bancos de dados relacionais padrão, o que significa que consultas complexas nativas do MySQL ou Postgres de Produção podem não funcionar nele, criando uma "falsa segurança" de que seu teste passou.

A solução profissional da indústria é usar a biblioteca **Testcontainers**. Ela sobe instâncias temporárias de sistemas (como Bancos de Dados ou o Kafka) baseadas em **Docker** puramente via código Java, executando os testes contra bancos de produção reais, e os destruindo ao final para não deixar lixo no sistema.

## 1. Dependências Maven Essenciais (Para o Spring Boot 3+)
Adicione o núcleo de testcontainers do Spring, o suporte a JUnit, e o driver específico do banco que desejar (aqui usaremos MySQL como base):

```xml
<!-- Testcontainers API & Integrações de Teste do Spring -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <scope>test</scope>
</dependency>

<!-- O módulo que trará o Banco de Dados exato de Produção que você quer emular -->
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>mysql</artifactId> <!-- Ou postgresql, oracle-free, kafka... -->
    <scope>test</scope>
</dependency>
```

## 2. A Mágica na Classe de Teste: `@ServiceConnection`
O Docker sobe os contêineres mapeando as portas padrão para **portas livres aleatórias** na sua máquina para evitar conflitos. Antigamente, conectar o Spring dinamicamente à porta que o Docker decidisse sortear exigia a injeção manual das rotas no arquivo `.properties` (via `@DynamicPropertySource`).

Com a chegada da funcionalidade **`@ServiceConnection`**, a inteligência do Spring entende o contêiner e **substitui automaticamente** a URL, Username e Password da fonte de dados pra você na inicialização. Magia pura!

```java
// 1. Define que os testes do Spring irão engatilhar o ciclo de vida do Testcontainers 
@Testcontainers
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserIntegrationTest {

    // 2. A anotação @Container permite ao JUnit matar o contêiner após os testes.
    // 3. A anotação @ServiceConnection substitui a URL do Spring Data apontando pro Docker gerado.
    // DICA DE OURO: Sempre informe a versão exata do banco de produção (ex: 8.4.0) em vez da "latest", evitando surpresas na esteira de deploy.
    @Container
    @ServiceConnection 
    private static MySQLContainer<?> mysqlContainer = new MySQLContainer<>("mysql:8.4.0");

    @Test
    void garanteQueOContainerFoiSubidoNaMemoria() {
        assertTrue(mysqlContainer.isCreated());
        assertTrue(mysqlContainer.isRunning());
    }
}
```

## 3. Resolvendo Erros de Sincronia na Inicialização ("Mapped port can only be obtained...")
Quando organizamos testes em ordem de dependência (como foi ensinado no Capítulo 11: 1. Salvar Usuário, 2. Logar, 3. Buscar os Dados) juntando com a exigência do ciclo de vida único da classe (`@TestInstance(PER_CLASS)`), podemos enfrentar um "Race Condition". O Spring Boot pode tentar conectar ao banco antes do Docker relatar que ele já terminou seu boot.

A solução que o professor propõe para contornar isso e "forçar" o MySQL a startar completamente antes do servidor do Spring iniciar é usar o bloco estático tradicional do Java (`static { }`) para engatilhar o arranque manual do contêiner.

```java
@Testcontainers
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class E2EWithDockerTest {

    // APAGAMOS O @Container pois vamos assumir o comando de iniciar
    @ServiceConnection
    private static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.4.0");

    // O bloco estático é lido pelo Java e acionado antes de qualquer injeção do Spring 
    static {
        mysql.start(); 
    }
    
    // ... os @Test de Logins e Autenticação fluem tranquilamente
}
```

## 4. E outros Ambientes? (Kafka, Oracle, Postgres)
O padrão de importação é sempre o mesmo. Se o seu projeto precisa usar um serviço diferente, basta alterar a dependência Maven do módulo no Pom.xml e iniciar a classe do contêiner correspondente:
*   **PostgreSQL:** `private static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:latest");`
*   **Oracle Free:** `private static OracleContainer oracle = new OracleContainer("gvenzl/oracle-free:23.4-slim-faststart");`
*   **Apache Kafka:** `private static KafkaContainer kafka = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:latest"));`
