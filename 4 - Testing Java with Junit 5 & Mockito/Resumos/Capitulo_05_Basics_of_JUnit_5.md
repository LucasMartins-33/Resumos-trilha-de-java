# Capítulo 05: Fundamentos do JUnit 5 (Basics)

Neste capítulo são abordadas as anotações essenciais, a construção e a otimização de testes usando as ferramentas fundamentais fornecidas pela API do JUnit Jupiter.

## 1. Estrutura de Código: Arrange, Act, Assert (AAA)
A estrutura ideal de um método de teste deve ser dividida em três partes claras (também conhecidas no formato BDD como *Given, When, Then*):
*   **Arrange (Preparação):** Você declara variáveis de entrada, resultados esperados e instâncias. Ex: `int dividend = 4; int divisor = 2; int expected = 2;`
*   **Act (Ação):** O método real a ser testado é invocado. Ex: `int actual = calc.integerDivision(dividend, divisor);`
*   **Assert (Validação):** O resultado obtido é validado com o resultado esperado. Ex: `assertEquals(expected, actual);`

## 2. Asserções e Mensagens Preguiçosas (Lazy Messages)
A classe estática `org.junit.jupiter.api.Assertions` fornece métodos utilitários para fazer as verificações, sendo os mais comuns:
*   `assertEquals(esperado, obtido, "Msg erro")`
*   `assertNotEquals(...)`
*   `assertTrue(condicao)` e `assertFalse(condicao)`
*   `assertNull(...)` e `assertNotNull(...)`

**🚀 Dica de Performance (Lazy Assert Messages):**
Se a sua mensagem de erro precisar concatenar muitas variáveis (ex: `"Erro. O valor " + a + " não bate com " + b`), isso consumirá memória e processamento *mesmo que o teste passe*. 
Para otimizar isso, o JUnit 5 permite passar a mensagem dentro de uma **expressão Lambda** vazia `() -> "Sua mensagem"`. Assim, a String só será processada dinamicamente caso o teste venha a falhar:
```java
assertEquals(esperado, resultado, () -> "O cálculo falhou para " + a + " e " + b);
```

## 3. Nomenclatura dos Testes e `@DisplayName`
Uma das convenções famosas para nomear métodos de teste é: `test[MetodoTestado]_[CondicaoOuEstado]_[ResultadoEsperado]`. 
*   Exemplo: `testIntegerDivision_WhenDividendIsDividedByZero_ShouldThrowArithmeticException()`

Como nomes assim são péssimos de ler em relatórios do terminal, usamos a anotação **`@DisplayName`** no nível da classe ou método para dar apelidos amigáveis:
```java
@DisplayName("Divisão: Deve lançar exceção ao tentar dividir por zero")
@Test
void testIntegerDivision_WhenDividendIsDividedByZero_ShouldThrowArithmeticException() { ... }
```

## 4. Ciclo de Vida do Teste (Lifecycle Annotations)
Por padrão, a ordem de execução dos métodos de teste é aleatória. Para configurar estados ou "limpar o terreno" usamos anotações de ciclo de vida:

*   **`@BeforeAll` (Antes de Todos):** Roda **1 única vez**, antes da classe inteira. Obrigatório que o método seja estático (`static`). Útil para subir um banco de dados em memória, por exemplo.
*   **`@BeforeEach` (Antes de Cada):** Roda **antes de cada** `@Test`. É o local mais usado para inicializar a classe que será testada (`calculadora = new Calculadora()`). Isso garante que cada teste terá uma calculadora "limpa", sem restos de informações de um teste anterior.
*   **`@AfterEach` (Depois de Cada):** Roda **após cada** teste terminar. Útil para apagar dados de tabelas.
*   **`@AfterAll` (Depois de Todos):** Roda **1 única vez**, no final de tudo. Obrigatório ser estático (`static`). Fechamento de conexões ocorrem muito aqui.

## 5. Como Desabilitar um Teste (@Disabled)
Para ignorar temporariamente um teste quebrado, **NÃO apenas apague o `@Test`**. Isso fará o teste sumir do mapa. Em vez disso, adicione a anotação **`@Disabled`**:
```java
@Disabled("Motivo: Método de pagamento ainda está fora do ar. Arrumar na sprint 2.")
@Test
void testaAprovacaoPagamento() { ... }
```
Assim, ele aparecerá nos relatórios como "Pulado (Skipped)", lembrando você de consertá-lo.

## 6. Testando Exceções (`assertThrows`)
Quando a regra de negócio **exige** que uma exceção estoure (ex: proibido dividir por zero ou processar idade negativa), o teste deve focar em validar se a exceção correta realmente apareceu.
Usamos o `assertThrows`:

```java
@DisplayName("Square root of negative number")
@Test
void testSquareRoot_WhenNumberIsNegative_ShouldThrowIllegalArgumentException() {
    // Arrange
    double negativeNum = -9.0;
    
    // Act & Assert acoplados (A ação acontece DENTRO do Assert)
    // Passamos 1º a Classe da Exceção esperada, e 2º um lambda executando o código falho
    IllegalArgumentException exceptionCapturada = assertThrows(
        IllegalArgumentException.class, 
        () -> { calculator.squareRoot(negativeNum); }, 
        "Deveria ter lançado IllegalArgumentException"
    );
    
    // Como a variável guarda o objeto da exceção, podemos assertar a mensagem gerada!
    assertEquals("Cannot calculate square root of negative number", exceptionCapturada.getMessage());
}
```
