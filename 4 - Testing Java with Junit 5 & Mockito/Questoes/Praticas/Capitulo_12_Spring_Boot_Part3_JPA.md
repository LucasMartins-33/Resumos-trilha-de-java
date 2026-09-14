# Questões Práticas - Capítulo 12 (Spring Boot Parte 3 - Testando a Camada de Dados - JPA Entities)

---

### 🟢 Nível 1: Configurando o Teste Fatiado de Persistência com `@DataJpaTest`
**Cenário:** Você precisa testar os mapeamentos da entidade `UserEntity` sem subir controladores web ou serviços de negócio.
**Sua Tarefa:**
* Crie a classe de teste `UserEntityTest`.
* Anote a classe com `@DataJpaTest`.
* Injete o gerenciador de entidades de teste fornecido pelo Spring:
  `@Autowired private TestEntityManager testEntityManager;`.
* Crie um método `@Test` inicial e comprove que o Spring sobe o contexto de persistência em memória (H2) rapidamente.

---

### 🟡 Nível 2: O Teste de "Caminho Feliz" (Happy Path) da Entidade
**Cenário:** Você precisa garantir que uma entidade preenchida com valores válidos seja persistida com sucesso e receba um ID gerado pelo banco.
**Sua Tarefa:**
* Instancie um `UserEntity user = new UserEntity();`.
* Preencha todos os campos obrigatórios: `setFirstName("Lucas")`, `setLastName("Silva")`, `setEmail("lucas@teste.com")`, `setUserId("UID-123")`.
* Persista o objeto usando `UserEntity salvo = testEntityManager.persistAndFlush(user);`.
* Assere com JUnit 5 que `salvo.getId() > 0` (chave primária auto-incrementada gerada pelo banco).

---

### 🟠 Nível 3: Comprovando o Rollback Transacional Automático
**Cenário:** Você quer comprovar que o `@DataJpaTest` mantém o banco de dados limpo e isolado entre métodos de teste sem deixar lixo residual.
**Sua Tarefa:**
* No método `testeA()`, persista um usuário com o e-mail `"unico@email.com"` usando `persistAndFlush`.
* No método `testeB()`, faça uma busca ou conte a quantidade de registros no banco via `testEntityManager.getEntityManager()`.
* Verifique se o registro inserido no `testeA` desapareceu no `testeB`, comprovando o rollback automático ao final de cada método.

---

### 🔴 Nível 4: A Importância do `flush()` vs Apenas `persist()`
**Cenário:** Um desenvolvedor escreveu um teste de validação de banco chamando apenas `testEntityManager.persist(user)` e o teste não disparou nenhuma exceção, mesmo com dados inválidos.
**Sua Tarefa:**
* Explique o conceito de *write-behind* (escrita postergada) do Hibernate/JPA.
* Por que a chamada de `persistAndFlush()` é estritamente necessária quando desejamos capturar erros e violações de constraints de banco de dados nos testes?

---

### 🟣 Nível 5: Testando a Restrição de Tamanho Máximo (`length = 50`)
**Cenário:** A coluna `first_name` está anotada com `@Column(nullable = false, length = 50)`. Textos com mais de 50 caracteres devem ser rejeitados pelo banco.
**Sua Tarefa:**
* Crie uma string contendo mais de 50 caracteres (ex: 55 caracteres de `"A"`).
* Instancie a entidade atribuindo essa string ao atributo `firstName`.
* Utilize `assertThrows(PersistenceException.class, () -> { testEntityManager.persistAndFlush(user); });`.
* Comprove que a tentativa de inserção física no banco dispara a violação de tamanho de coluna.

---

### 🟤 Nível 6: Testando a Restrição de Nulidade (`nullable = false`)
**Cenário:** O campo `email` é obrigatório no modelo relacional e possui a anotação `@Column(nullable = false)`.
**Sua Tarefa:**
* Instancie `UserEntity` com dados válidos, mas deixe o atributo `email` como `null`.
* Tente persistir com `testEntityManager.persistAndFlush(user)` dentro de um `assertThrows`.
* Capture a `PersistenceException` e confirme que a restrição de nulidade do banco de dados impediu a gravação do registro.

---

### 🔵 Nível 7: Testando a Restrição de Chave Única (`unique = true`)
**Cenário:** Dois usuários diferentes não podem possuir o mesmo `userId` no sistema (`@Column(unique = true)`).
**Sua Tarefa:**
* Na etapa Arrange, instancie o primeiro usuário com `setUserId("ID-DUPLICADO")` e execute `testEntityManager.persistAndFlush(user1);` com sucesso.
* Instancie um segundo usuário diferente, atribuindo a ele o mesmo `setUserId("ID-DUPLICADO")`.
* Na etapa Act & Assert, envolva a chamada `testEntityManager.persistAndFlush(user2)` em um `assertThrows(PersistenceException.class, ...)`.
* Confirme que o banco dispara violação de constraint de integridade única.

---

### 🟢 Nível 8: Diferenciando Erros de `@Valid` de Erros de `@Column`
**Cenário:** O time de segurança quer entender a diferença entre proteções de entrada da API e proteções físicas do banco de dados.
**Sua Tarefa:**
* Descreva em um comentário estruturado na classe de teste:
  * O que acontece se a API receber um campo inválido e o controller estiver anotado com `@Valid` (qual status HTTP é devolvido)?
  * O que acontece se a validação da API falhar em filtrar o dado e a entidade tentar ser gravada diretamente no banco violando uma `@Column`?
* Por que as restrições na entidade são consideradas a "última linha de defesa" da integridade dos dados?

---

### 🟡 Nível 9: Inspecionando Consultas com `testEntityManager.find()`
**Cenário:** Após persistir uma entidade, você precisa simular uma consulta limpa direto do banco garantindo que os dados não vieram do cache da sessão.
**Sua Tarefa:**
* Salve a entidade com `testEntityManager.persistAndFlush(user);`.
* Chame `testEntityManager.clear();` para limpar o cache de primeiro nível do Hibernate.
* Realize a busca do usuário pelo ID gerado: `UserEntity encontrado = testEntityManager.find(UserEntity.class, user.getId());`.
* Valide com `assertEquals` que todos os campos foram recuperados com exatidão da tabela física.

---

### 🟠 Nível 10: Integração Final (Validação Exaustiva da Entidade de Domínio)
**Cenário:** Você precisa entregar uma suíte completa de testes para a entidade `ProdutoEntity(id, codigoBarras, nome, preco, dataCriacao)`.
**Sua Tarefa:**
* Escreva uma classe `ProdutoEntityTest` anotada com `@DataJpaTest`.
* Cubra exaustivamente com testes unitários/integrados:
  1. **Caminho Feliz:** Produto válido persistido com ID > 0.
  2. **Tamanho de Nome:** Nome com mais de 100 caracteres dispara `PersistenceException`.
  3. **Código de Barras Único:** Dois produtos com o mesmo código de barras disparam erro de duplicidade.
  4. **Preço Não Nulo:** Produto com preço nulo viola constraint de nulidade.
  5. **Data de Criação Padrão:** Valide se campos automáticos (`@CreationTimestamp` ou valores default) são preenchidos corretamente após o flush.
