# Questões Práticas - Capítulo 13 (Testando JPA Repositories)

---

### 🟢 Nível 1: Identificando o que NÃO Testar
**Cenário:** Um desenvolvedor júnior escreveu 15 testes unitários apenas para chamar `usersRepository.save()`, `usersRepository.findById()` e `usersRepository.deleteById()`.
**Sua Tarefa:**
* Explique por que escrever testes automatizados para métodos pré-fabricados herdados do `JpaRepository` é considerado um desperdício de tempo e esforço.
* Quais são os únicos dois tipos de métodos em interfaces de repositório que realmente justificam a criação de testes automatizados?

---

### 🟡 Nível 2: Preparando a Infraestrutura de Teste do Repositório
**Cenário:** Você precisa preparar o ambiente de teste integrado para a interface `UsersRepository`.
**Sua Tarefa:**
* Crie a classe de teste `UsersRepositoryTest`.
* Anote a classe com `@DataJpaTest`.
* Injete a ferramenta para preparação de dados de cobaia:
  `@Autowired private TestEntityManager testEntityManager;`.
* Injete a interface real sob teste:
  `@Autowired private UsersRepository usersRepository;`.

---

### 🟠 Nível 3: Testando um Query Method Simples (`findByEmail`)
**Cenário:** Você declarou na sua interface o método `UserEntity findByEmail(String email);` e quer comprovar que a consulta automática gerada pelo Spring Data funciona.
**Sua Tarefa:**
* Na etapa Arrange, instancie um usuário com e-mail `"lucas@empresa.com"` e execute `testEntityManager.persistAndFlush(user);`.
* Na etapa Act, acione a busca: `UserEntity encontrado = usersRepository.findByEmail("lucas@empresa.com");`.
* Na etapa Assert, certifique-se com `assertNotNull(encontrado)` e valide se o e-mail retornado é estritamente `"lucas@empresa.com"`.

---

### 🔴 Nível 4: Validando Resultados Negativos (Registro Inexistente)
**Cenário:** O repositório precisa se comportar de maneira previsível quando o critério de busca não corresponder a nenhum registro no banco.
**Sua Tarefa:**
* No teste `testFindByEmail_QuandoNaoExistir_DeveRetornarNull()`, limpe a base ou não insira registros correspondentes.
* Chame `usersRepository.findByEmail("inexistente@email.com");`.
* Valide com `assertNull(encontrado)` ou `assertTrue(optionalUser.isEmpty())` caso o método retorne `Optional<UserEntity>`.

---

### 🟣 Nível 5: Testando Queries Derivadas com Múltiplos Parâmetros
**Cenário:** Você criou o método `List<UserEntity> findByFirstNameAndLastName(String firstName, String lastName);`.
**Sua Tarefa:**
* Na etapa Arrange, persista três registros cobaia:
  1. Lucas Silva
  2. Lucas Souza
  3. Marcos Silva
* Na etapa Act, busque por `("Lucas", "Silva")`.
* Na etapa Assert, valide se a lista retornada possui tamanho 1 (`assertEquals(1, lista.size())`) e que o item possui os dados exatos.

---

### 🟤 Nível 6: A Importância dos Dados Mistos em Filtros de Lista
**Cenário:** Você criou um método com `@Query("SELECT u FROM UserEntity u WHERE u.email LIKE %:domain")` para buscar usuários por provedor de e-mail.
**Sua Tarefa:**
* Explique por que inserir apenas usuários do `@gmail.com` na etapa Arrange seria um teste frágil e insuficiente.
* Implemente o teste inserindo um usuário com `@gmail.com` e outro com `@hotmail.com`.
* Execute a consulta buscando por `"@gmail.com"` e valide se apenas o usuário correto foi incluído no resultado (`size() == 1`).

---

### 🔵 Nível 7: Testando Consultas Customizadas com Parâmetros Nomeados (`@Param`)
**Cenário:** Na interface, você declarou uma consulta JPQL com parâmetro nomeado:
```java
@Query("SELECT u FROM UserEntity u WHERE u.firstName = :nome AND u.status = :status")
List<UserEntity> buscarPorNomeEStatus(@Param("nome") String nome, @Param("status") String status);
```
**Sua Tarefa:**
* Crie o método de teste correspondente no `UsersRepositoryTest`.
* Popule a base com dados que satisfaçam ambos os filtros e dados que satisfaçam apenas um dos filtros (ex: mesmo nome com status inativo).
* Assere que a consulta JPQL compila sem erros de sintaxe de HQL/JPQL e retorna apenas registros com status ativo.

---

### 🟢 Nível 8: Testando Consultas Nativas de Banco (`nativeQuery = true`)
**Cenário:** Por razões de performance, você utilizou SQL puro do banco de dados relacional:
```java
@Query(value = "SELECT * FROM users u WHERE u.created_at >= CURRENT_DATE - 7", nativeQuery = true)
List<UserEntity> buscarUsuariosRecentes();
```
**Sua Tarefa:**
* Escreva um teste automatizado para essa consulta nativa.
* Persista um usuário com data de criação de ontem e outro com data de criação de 30 dias atrás.
* Invoque o método e comprove que o SQL nativo executa corretamente sobre a base embutida e retorna apenas o usuário recente.

---

### 🟡 Nível 9: Isolando o Repositório de Camadas Superiores
**Cenário:** Um colega sugeriu injetar `@Autowired private UserService userService;` dentro do `UsersRepositoryTest` para cadastrar os usuários de teste.
**Sua Tarefa:**
* Explique por que injetar o serviço dentro do teste de repositório viola o isolamento e a velocidade do teste fatiado (`@DataJpaTest`).
* Qual ferramenta nativa do Spring deve ser usada para manipular dados preparatórios nos testes da camada de dados sem acoplar a classe de serviço?

---

### 🟠 Nível 10: Integração Final (Suíte de Repositório de Pedidos com Relatórios)
**Cenário:** Você precisa entregar a suíte de testes completa do repositório `PedidosRepository` contendo regras de busca analíticas complexas.
**Sua Tarefa:**
* O repositório possui:
  * Um Query Method derivado: `findByClienteIdAndStatus(Long clienteId, StatusPedido status)`.
  * Uma consulta JPQL customizada: busca o valor total somado de pedidos de um cliente (`SELECT SUM(p.valorTotal) FROM Pedido p WHERE p.clienteId = :clienteId`).
* Na classe de testes:
  * Use `@DataJpaTest` com `TestEntityManager`.
  * Popule dados mistos (pedidos de clientes diferentes e com status cancelados vs aprovados).
  * Valide a busca com filtros compostos.
  * Valide se o cálculo do somatório bate exatamente com a soma matemática dos pedidos aprovados.
