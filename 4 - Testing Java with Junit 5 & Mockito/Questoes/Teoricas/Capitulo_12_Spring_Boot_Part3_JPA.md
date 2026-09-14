# Questões Teóricas - Capítulo 12: Spring Boot (Parte 3) - Testando a Camada de Dados (JPA Entities)

Testes de fixação sobre testes fatiados de persistência com `@DataJpaTest`, uso de `TestEntityManager`, transações com rollback e validação de constraints físicas de banco.

---

### 1. Qual é o escopo e o propósito da anotação `@DataJpaTest` no ecossistema de testes do Spring Boot?

> [!faq]- 👀 Ver Resposta
> A anotação `@DataJpaTest` configura um teste fatiado voltado exclusivamente para a camada de persistência (Data Layer). Ela inicializa apenas os componentes essenciais para o funcionamento do JPA e Hibernate (como entidades `@Entity`, repositórios do Spring Data, `DataSource` e o gerenciador de transações), ignorando completamente controladores web (`@Controller`), classes de serviço (`@Service`) e componentes de infraestrutura desnecessários, resultando em testes leves e focados.

---

### 2. Como a anotação `@DataJpaTest` gerencia a fonte de dados (banco de dados) durante a execução dos testes por padrão?

> [!faq]- 👀 Ver Resposta
> Por padrão, ao detectar a anotação `@DataJpaTest`, o Spring Boot substitui automaticamente o banco de dados principal de produção por uma instância embutida em memória (como H2, HSQL ou Derby), caso a dependência esteja presente no classpath. Isso permite que os testes executem de forma autocontida e imediata, sem exigir que um banco de dados relacional externo esteja instalado e rodando na máquina.

---

### 3. Como funciona a política transacional padrão do `@DataJpaTest` e por que ela é benéfica para a repetibilidade dos testes?

> [!faq]- 👀 Ver Resposta
> Por padrão, todos os métodos de teste dentro de uma classe anotada com `@DataJpaTest` são anotados implicitamente com `@Transactional`. Isso significa que o teste executa dentro de uma transação aberta do banco de dados e, assim que o método termina (seja por sucesso ou falha), o Spring automaticamente dispara um **Rollback**. Todas as inserções, atualizações e deleções feitas pelo teste são desfeitas, garantindo que o banco de dados permaneça limpo e isolado para o próximo teste, preservando o princípio da independência.

---

### 4. O que é o utilitário `TestEntityManager` e qual a sua vantagem sobre o `EntityManager` padrão do JPA em testes?

> [!faq]- 👀 Ver Resposta
> O `TestEntityManager` é uma subclasse utilitária fornecida pelo Spring Boot Test que encapsula o `EntityManager` tradicional do JPA com métodos desenhados especificamente para cenários de teste. Ele oferece operações combinadas e convenientes — como `persistAndFlush()`, `persistFlushFind()` e `clear()` —, facilitando a sincronização imediata de dados com a tabela de teste sem a necessidade de gerenciar sessões e transações de forma verbosa.

---

### 5. Por que faz sentido escrever testes automatizados para classes de Entidade JPA se elas aparentemente contêm apenas atributos, getters e setters?

> [!faq]- 👀 Ver Resposta
> Porque as classes de entidade contêm os mapeamentos objeto-relacional (ORM) e as restrições físicas de integridade do banco de dados (constraints) definidas por anotações como `@Column(nullable = false, length = 50, unique = true)`. Testar a entidade é a forma mais eficaz de assegurar que essas barreiras de validação física configuradas nas anotações são de fato aplicadas pelo mecanismo de persistência e que o banco rejeitará dados corrompidos ou ilegais.

---

### 6. Por que o método `testEntityManager.persistAndFlush()` é utilizado em testes de restrições de entidade em vez de apenas `persist()`?

> [!faq]- 👀 Ver Resposta
> O JPA (Hibernate) utiliza o conceito de *write-behind* (escrita postergada): ao chamar apenas `persist()`, a entidade é colocada no contexto de persistência em memória (cache de primeiro nível) e o comando SQL `INSERT` não é enviado imediatamente ao banco. O método `persistAndFlush()` força a sincronização física imediata do cache com o banco de dados naquele exato momento (`FLUSH`). É somente nesse instante que as restrições do banco (tamanho de coluna, nulidade, chaves únicas) são avaliadas e disparam as exceções que o teste deseja capturar.

---

### 7. O que acontece quando se tenta persistir uma entidade violando a restrição de tamanho máximo definida por `@Column(length = 50)`?

> [!faq]- 👀 Ver Resposta
> Quando a sincronização (`flush`) ocorre, o motor de persistência (Hibernate) ou o próprio banco de dados relacional rejeita a inserção devido à tentativa de gravar uma cadeia de caracteres superior à largura definida para a coluna. A operação aborta e lança uma exceção da família de persistência, normalmente encapsulada pelo JPA em uma `jakarta.persistence.PersistenceException` (ou subclasses específicas do Hibernate como `DataException`), comprovando a atuação da barreira.

---

### 8. Como deve ser desenhado um teste para validar a restrição de unicidade (`unique = true`) de uma coluna (por exemplo, `userId` ou `email`)?

> [!faq]- 👀 Ver Resposta
> O teste deve ser construído em três passos:
> 1. **Arrange:** Cria uma primeira entidade com o identificador único (ex: `email = "teste@email.com"`) e a grava fisicamente no banco com `persistAndFlush()` (operação que deve passar com sucesso).
> 2. **Arrange:** Instancia uma segunda entidade distinta, atribuindo a ela o mesmo identificador duplicado (`email = "teste@email.com"`).
> 3. **Act & Assert:** Executa a persistência da segunda entidade com `persistAndFlush()` dentro de um `assertThrows(PersistenceException.class, ...)`, confirmando que o banco recusa a inserção e aciona a violação de integridade.

---

### 9. Qual é a importância de incluir um teste de "Caminho Feliz" (Happy Path) para a persistência de entidades JPA?

> [!faq]- 👀 Ver Resposta
> Enquanto os testes negativos garantem que dados inválidos são barrados, o teste de caminho feliz comprova que uma entidade configurada com dados 100% corretos consegue ser persistida com êxito. Além disso, ele permite asserir comportamentos automáticos do banco, como a geração correta de identificadores primários auto-incrementados anotados com `@GeneratedValue` (`assertTrue(user.getId() > 0)`), garantindo que o ciclo de vida da entidade funciona de ponta a ponta.

---

### 10. Qual a diferença fundamental entre as validações originadas por anotações `@Column` do JPA e validações originadas por anotações do Bean Validation (`@Valid` / `@NotBlank`)?

> [!faq]- 👀 Ver Resposta
> * As anotações do **Bean Validation** (`@NotBlank`, `@Size`, `@Email`) atuam nas camadas superiores (como na entrada de Controllers) antes que os dados alcancem as regras de persistência. Quando falham, disparam exceções tratáveis que resultam em códigos amigáveis ao cliente HTTP (`400 Bad Request`).
> * As anotações de **`@Column` do JPA** (`length`, `nullable`, `unique`) atuam como a última barreira de proteção no banco de dados. Quando violadas, resultam em erros catastróficos de banco (`PersistenceException` / SQL Errors), projetados para impedir corrupção física do banco caso as barreiras das camadas superiores falhem.
