# Capítulo 12: Spring Boot (Parte 3) - Testando a Camada de Dados (JPA Entities)

Neste capítulo o foco foi descer para a camada mais profunda da aplicação (Data Layer) e aprender a testar as lógicas do banco de dados e os mapeamentos do JPA (Hibernate) de forma totalmente isolada das demais partes do sistema (Controllers e Services não são carregados).

## 1. A Anotação `@DataJpaTest`
Para subir uma "fatia" do Spring focada apenas na comunicação com o banco de dados, utilizamos o `@DataJpaTest` no topo da nossa classe de testes.
Essa anotação possui "poderes" embutidos que facilitam muito a nossa vida:
*   **In-Memory Database:** Ela detecta que é um ambiente de testes e sobe automaticamente um banco de dados em memória (como o H2) para testarmos, poupando configuração manual de conexões.
*   **Filtro de Contexto:** Cria apenas Beans como Entidades e Repositórios. Se o seu serviço de e-mail estiver mal configurado, não vai quebrar o teste, pois ele nem sequer tentará ser lido.
*   **Rollback Automático (Transacional):** Cada método `@Test` é executado dentro de uma transação do banco. Assim que o método de teste encerra, o Spring faz um *rollback*, apagando os dados que foram inseridos e deixando a tabela limpa para o próximo teste rodar sem lixo residual.

## 2. A Ferramenta `TestEntityManager`
Para testarmos as validações estruturais das nossas Tabelas e Entidades (salvar, consultar e checar lógicas), não precisamos criar os Repositórios antecipadamente. O Spring nos fornece um gerenciador otimizado para cenários de testes: o **`TestEntityManager`**.

```java
@DataJpaTest
class UserEntityTest {

    @Autowired // O Spring injeta magicamente a ferramenta pra nós
    private TestEntityManager testEntityManager;
    
    // ...
}
```

## 3. Garantindo Restrições de Tabela (Constraints)
Por que nós escreveríamos um teste para uma classe burra de Entidade (`UserEntity.java`)?
Porque é nessa classe que as restrições físicas do Banco de Dados moram, através da anotação `@Column`. Precisamos testar se elas estão aplicando as barreiras que desejamos (Ex: não aceitar um ID que já existe e não estourar o tamanho do campo Varchar no banco).

Usamos o método `testEntityManager.persistAndFlush()` para forçar a inserção física imediata no banco, obrigando-o a disparar exceções caso a operação infrinja as regras mapeadas.

### Exemplo 1: Tamanho Máximo (`length = 50`)
Garante que a constraint de tamanho máximo do campo seja respeitada e o banco bloqueie a inserção de textos exagerados.
```java
@Test
void deveLancarExcecaoQuandoFirstNameForMaiorQueAColunaPermite() {
    UserEntity user = new UserEntity();
    user.setFirstName("NomeMuitoGrandeQueEstouraOLimitePermitidoDaAnotacaoColumnNaEntidade");
    
    // Como a configuração diz @Column(length=50), o banco tem que estourar um erro ao dar o Flush
    assertThrows(PersistenceException.class, () -> {
        testEntityManager.persistAndFlush(user);
    });
}
```

### Exemplo 2: Evitando Registros Duplicados (`unique = true`)
Valida se a regra de que dois usuários não podem ter o mesmo "UserId" ou o mesmo "Email" está funcionando.
```java
@Test
void deveLancarExcecaoSeSalvarUserIdDuplicado() {
    // 1. Arrange: Cria e salva o PRIMEIRO user com id "CODIGO-123"
    UserEntity user1 = new UserEntity();
    user1.setUserId("CODIGO-123");
    testEntityManager.persistAndFlush(user1); // Sucesso!
    
    // 2. Arrange: Tenta criar o SEGUNDO user com o id clonado "CODIGO-123"
    UserEntity user2 = new UserEntity();
    user2.setUserId("CODIGO-123");
    
    // 3. Act & Assert: Ao dar o flush, a constraint Unique precisa disparar a exceção
    assertThrows(PersistenceException.class, () -> {
        testEntityManager.persistAndFlush(user2);
    });
}
```

### A Importância do Caminho Feliz
Além de testar as falhas, é essencial sempre ter um teste provando que, ao passar os dados 100% validados, a inserção flui corretamente e que campos anotados com `@GeneratedValue` (`id` auto-increment numérico) realmente recebam um número do banco superior a zero!

> [!TIP]
> **Diferenciando Barreiras:** Lembre-se que as proteções de `@Column` geram erros de banco de dados (`PersistenceException`) para manter a integridade dos dados, operando de forma diferente das validações de Input de API (`@Valid`), que devolvem erros amigáveis ao usuário (`400 Bad Request`). Ambas são igualmente importantes na segurança da aplicação!
