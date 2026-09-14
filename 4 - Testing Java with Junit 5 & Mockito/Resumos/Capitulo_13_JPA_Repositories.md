# Capítulo 13: Testando JPA Repositories

Ao testar as interfaces de Repositório do Spring Data JPA, é essencial entender o limite entre o nosso código e o código do framework para não perder tempo escrevendo testes redundantes.

## 1. Regra de Ouro: O que NÃO Testar
Nós **não** devemos testar os métodos prontos herdados pelo `CrudRepository` ou `JpaRepository`, como:
*   `save()`
*   `delete()`
*   `findAll()`
*   `findById()`

A equipe do Spring já escreveu milhares de testes para essas funções. Confie no framework.

## 2. O Que DEVE Ser Testado
*   **Query Methods (Consultas Derivadas por Nome):** São os métodos que você nomeia seguindo o padrão do Spring (ex: `findByEmailAndFirstName`). Como o Spring usa inteligência para extrair as instruções SQL do próprio nome do método, se você errar a nomenclatura, a consulta sairá errada.
*   **Consultas com `@Query` (JPQL ou Nativas):** Sempre que você escreve o script SQL / JPQL na mão dentro da anotação `@Query`, você assume o risco. É fundamental escrever testes para garantir que aquele SQL está realmente filtrando as informações certas no banco.

## 3. Preparando a Classe de Teste
A mesma anotação do capítulo anterior é utilizada: **`@DataJpaTest`**. Com ela ativamos o banco em memória e os testes transacionais.

```java
@DataJpaTest
class UsersRepositoryTest {

    // Ferramenta usada na etapa ARRANGE (preparar a massa de dados falsos)
    @Autowired
    private TestEntityManager testEntityManager;

    // A classe REAL que queremos testar (Método ACT)
    @Autowired
    private UsersRepository usersRepository;
    
    // ...
}
```

## 4. Testando um Query Method Simples
Vamos testar um método chamado `findByEmail`.
```java
@Test
void deveEncontrarUmUsuarioBaseadoNoEmail() {
    // 1. Arrange: Insere fisicamente um registro na tabela para servir de cobaia
    UserEntity user = new UserEntity();
    user.setFirstName("Lucas");
    user.setEmail("lucas@teste.com");
    testEntityManager.persistAndFlush(user);

    // 2. Act: Aciona a consulta do seu Repositório passando o e-mail exato
    UserEntity storedUser = usersRepository.findByEmail("lucas@teste.com");

    // 3. Assert: Valida se encontrou e se de fato os dados estão corretos
    assertNotNull(storedUser);
    assertEquals("lucas@teste.com", storedUser.getEmail());
}
```

## 5. Testando Consultas Customizadas com `@Query` (JPQL)
Imagine que você possua o seguinte método no repositório:
```java
@Query("SELECT u FROM UserEntity u WHERE u.email LIKE %:domain")
List<UserEntity> findUsersWithEmailEndingWith(@Param("domain") String domain);
```
O ideal ao testar um cenário de "lista filtrada" é popular o banco de dados de cobaia com **resultados mistos** (dados que combinam e dados que não combinam com a condição), e provar que o SQL traz apenas os corretos.

```java
@Test
void deveRetornarUsuariosCujoEmailTerminaComOValorEspecificado() {
    // 1. Arrange: Insere um e-mail do Gmail
    UserEntity user1 = new UserEntity();
    user1.setEmail("lucas@gmail.com"); 
    testEntityManager.persistAndFlush(user1);

    // 1. Arrange: Insere um e-mail da Microsoft (que NÃO DEVE vir no SELECT)
    UserEntity user2 = new UserEntity();
    user2.setEmail("pedro@live.com"); 
    testEntityManager.persistAndFlush(user2);

    // 2. Act: Usa o repositório para buscar APENAS quem usa "@gmail.com"
    List<UserEntity> usersList = usersRepository.findUsersWithEmailEndingWith("@gmail.com");

    // 3. Assert: Garante que o JPQL filtrou certinho
    assertEquals(1, usersList.size(), "Havia 2 registros, mas a lista só deve trazer 1");
    assertTrue(usersList.get(0).getEmail().endsWith("@gmail.com"));
}
```
