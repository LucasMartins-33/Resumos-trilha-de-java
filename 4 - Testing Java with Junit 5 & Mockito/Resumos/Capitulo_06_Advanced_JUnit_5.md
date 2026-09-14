# Capítulo 06: Tópicos Avançados do JUnit 5

Este capítulo foca em recursos avançados para escrever testes mais dinâmicos (que reduzem a duplicação de código) e configurações robustas de ciclo de vida para testes de integração estatais (stateful).

## 1. Testes Parametrizados (`@ParameterizedTest`)
Quando precisamos testar a mesma lógica de negócio passando vários valores diferentes, copiar e colar o mesmo método `@Test` é uma má prática. O JUnit nos fornece o `@ParameterizedTest` para rodar o mesmo teste várias vezes usando diferentes injeções de dados.

Você deve substituir a anotação `@Test` por `@ParameterizedTest` e usar uma das fontes (*Sources*) abaixo:

### A) Valores Simples (`@ValueSource`)
Injeta um único array de valores em um teste que recebe **apenas 1 parâmetro**.
```java
@ParameterizedTest
@ValueSource(strings = {"John", "Kate", "Alice"})
void testaNomeNaoNulo(String firstName) {
    assertNotNull(firstName);
}
```

### B) Valores Compostos (`@CsvSource`)
Injeta vários parâmetros de uma vez, simulando colunas separadas por vírgulas.
```java
@ParameterizedTest
@CsvSource({
    "33, 1, 32", // Rodada 1: a=33, b=1, esperado=32
    "24, 1, 23", // Rodada 2
    "54, 1, 53"  // Rodada 3
})
void testSubtracao(int minuend, int subtrahend, int expectedResult) {
    assertEquals(expectedResult, calc.subtract(minuend, subtrahend));
}
```
*Dica para Strings: Use `''` para representar String Vazia e deixe vazio antes da vírgula para representar Nulo (`null`).*

### C) Arquivos Externos (`@CsvFileSource`)
Lê uma planilha `.csv` real salva na pasta `src/test/resources`. Excelente para massa de dados gigante.
```java
@ParameterizedTest
@CsvFileSource(resources = "/dados_da_subtracao.csv")
void testSubtracaoDoArquivo(int a, int b, int esperado) { ... }
```

### D) Geração via Código (`@MethodSource`)
A fonte de dados passa a ser um método estático customizado que retorna um Fluxo (`Stream`) de Argumentos. Muito útil para passar Objetos Complexos (`Listas`, Objetos customizados, instâncias mockadas) que não cabem num texto CSV.
```java
@ParameterizedTest
@MethodSource("geradorDeCenarios")
void testaComObjetos(int a, int b, int esperado) { ... }

private static Stream<Arguments> geradorDeCenarios() {
    return Stream.of(
        Arguments.of(33, 1, 32),
        Arguments.of(24, 1, 23)
    );
}
```

## 2. Repetindo Testes Seguidos (`@RepeatedTest`)
Para garantir que um método não falhe de forma intermitente (flaky test), você pode forçá-lo a rodar `X` vezes seguidas.
```java
@RepeatedTest(value = 3, name = "{displayName} - Repetição {currentRepetition} de {totalRepetitions}")
@DisplayName("Teste Estabilidade de Conexão")
void testaConexao() { ... }
```
*Se você precisar saber em qual repetição o teste está através do código, o JUnit injeta esses dados magicamente se você adicionar `RepetitionInfo info` e `TestInfo testInfo` nos argumentos do método de teste.*

## 3. Forçando a Ordem de Execução
Em testes unitários puros, a ordem de execução não deve importar. Mas em testes de integração, você pode querer controlar a ordem. Adicione a anotação `@TestMethodOrder` no topo da sua classe:

*   **`MethodOrderer.Random.class`**: Embaralha tudo. Bom para checar se seus testes estão com acoplamentos acidentais.
*   **`MethodOrderer.MethodName.class`**: Roda os testes na ordem alfabética do nome do método.
*   **`MethodOrderer.OrderAnnotation.class`**: Permite que você adicione `@Order(1)`, `@Order(2)` em cima de cada método e faça-os rodar na ordem numérica cravada.

> [!TIP]
> Também é possível forçar a **Ordem das Classes de Teste**. Basta criar um arquivo chamado `junit-platform.properties` na pasta `src/test/resources` com a linha: `junit.jupiter.testclass.order.default=org.junit.jupiter.api.ClassOrderer$OrderAnnotation`. Daí é só usar `@Order(1)` em cima do nome da classe que você quer que rode primeiro (ex: `UserTest`, seguido por `ProductTest`).

## 4. O Ciclo de Vida da Instância (`@TestInstance`)
Como aprendido no Capítulo 1 (F.I.R.S.T), testes devem ser *Independentes*. O JUnit garante isso **instanciando a classe inteira do zero para cada método `@Test` que ele encontra**. Ou seja, uma variável global (membro da classe) não mantém estado de um teste para o outro.

Mas em **Testes de Integração Estatais (Stateful)** — por exemplo: onde você cria um usuário no `Teste 1`, pega o ID gerado, joga numa variável global e usa esse mesmo ID para atualizar o usuário no `Teste 2` —, você precisará desligar esse comportamento.

Para isso, mudamos a anotação do Ciclo de Vida no topo da classe:
```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS) // Modifica para apenas UMA instância inteira
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class IntegracaoCRUDTest {
    String userId; // Agora a variável se manterá entre um teste e outro!

    @Test @Order(1)
    void criaUser() { this.userId = "123"; }

    @Test @Order(2)
    void deletaUser() { deletarNoBanco(this.userId); }
}
```

> [!NOTE]
> **Vantagem de Performance:** Ao usar o ciclo `PER_CLASS`, os métodos de setup e teardown (`@BeforeAll` e `@AfterAll`) **não precisam mais ter a assinatura `static`**, deixando o código muito mais limpo e amigável.
