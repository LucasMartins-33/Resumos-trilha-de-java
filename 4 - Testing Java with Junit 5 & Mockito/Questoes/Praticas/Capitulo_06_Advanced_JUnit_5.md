# Questões Práticas - Capítulo 06 (Tópicos Avançados do JUnit 5)

---

### 🟢 Nível 1: O Primeiro Teste Parametrizado com `@ValueSource`
**Cenário:** Você precisa testar se um validador de nomes rejeita strings nulas ou vazias para uma lista com 5 nomes válidos diferentes, sem duplicar o método de teste.
**Sua Tarefa:**
* Substitua a anotação `@Test` por `@ParameterizedTest`.
* Adicione a fonte `@ValueSource(strings = {"Ana", "Carlos", "Beatriz", "Daniel", "Eduarda"})`.
* Declare o parâmetro `String nome` na assinatura do método de teste.
* Valide com `assertNotNull(nome)` e `assertFalse(nome.trim().isEmpty())`.
* Observe a execução na IDE mostrando 5 rodadas individuais para o mesmo método.

---

### 🟡 Nível 2: Injeção Múltipla de Argumentos com `@CsvSource`
**Cenário:** Você quer testar uma operação de subtração matemática passando o minuendo, o subtraendo e o resultado esperado em várias rodadas.
**Sua Tarefa:**
* Crie um `@ParameterizedTest` utilizando a fonte `@CsvSource`.
* Forneça três cenários no array:
  ```java
  @CsvSource({
      "10, 3, 7",
      "25, 5, 20",
      "50, 50, 0"
  })
  ```
* Receba os três valores inteiros como parâmetros `(int a, int b, int esperado)`.
* Valide `assertEquals(esperado, calculadora.subtrair(a, b))`.

---

### 🟠 Nível 3: Tratando Strings Vazias e Nulas no `@CsvSource`
**Cenário:** Um método de sanitização de texto precisa ser testado contra valores especiais como strings vazias e valores nulos via CSV.
**Sua Tarefa:**
* Crie um `@ParameterizedTest` com `@CsvSource`:
  * Linha 1: `"Lucas, false"` (nome preenchido).
  * Linha 2: `'', true` (string vazia representada por aspas simples contíguas `''`).
  * Linha 3: `, true` (valor nulo omitindo o conteúdo antes da vírgula).
* No método `(String texto, boolean deveFalhar)`, valide o comportamento do seu sanitizador.

---

### 🔴 Nível 4: Lendo Dados Externos com `@CsvFileSource`
**Cenário:** A equipe de regras de negócio disponibilizou uma planilha com centenas de cálculos fiscais que deve alimentar seus testes unitários.
**Sua Tarefa:**
* Crie o arquivo `src/test/resources/dados_impostos.csv` contendo colunas de entrada e saída (ex: `100.0, 0.15, 15.0`).
* No seu método de teste, utilize `@ParameterizedTest` com `@CsvFileSource(resources = "/dados_impostos.csv")`.
* Receba os parâmetros do tipo `double` e valide a conformidade dos cálculos fiscais.

---

### 🟣 Nível 5: Fontes Avançadas com `@MethodSource` e `Arguments`
**Cenário:** Você precisa alimentar o teste parametrizado com instâncias completas de objetos `Usuario(id, perfil, permissoes)` que não podem ser representadas em CSV.
**Sua Tarefa:**
* Crie um `@ParameterizedTest` anotado com `@MethodSource("provedorDeUsuarios")`.
* Crie o método gerador estático:
  ```java
  private static Stream<Arguments> provedorDeUsuarios() {
      return Stream.of(
          Arguments.of(new Usuario("Lucas", "ADMIN"), true),
          Arguments.of(new Usuario("Joao", "GUEST"), false)
      );
  }
  ```
* No teste `(Usuario usuario, boolean podeAcessarPainel)`, valide a regra de segurança correspondente.

---

### 🟤 Nível 6: Repetindo Testes Instáveis com `@RepeatedTest`
**Cenário:** Você implementou um gerador de números aleatórios e quer garantir que, em 10 execuções consecutivas, nenhum número ultrapasse o limite superior de 100.
**Sua Tarefa:**
* Substitua `@Test` por `@RepeatedTest(value = 10, name = "{displayName} - Execução {currentRepetition} de {totalRepetitions}")`.
* Adicione os parâmetros `RepetitionInfo repetitionInfo` e `TestInfo testInfo` na assinatura do método de teste.
* No corpo do teste, imprima a repetição atual e assere que o número sorteado é menor ou igual a 100.

---

### 🔵 Nível 7: Ordenação Numérica de Testes com `@Order`
**Cenário:** Em um teste de integração de CRUD, é imperativo que a criação aconteça antes da consulta e da exclusão.
**Sua Tarefa:**
* No topo da classe de teste, adicione `@TestMethodOrder(MethodOrderer.OrderAnnotation.class)`.
* Crie os métodos:
  * `@Test @Order(1) void testCriarRegistro() { ... }`
  * `@Test @Order(2) void testConsultarRegistro() { ... }`
  * `@Test @Order(3) void testExcluirRegistro() { ... }`
* Execute a classe inteira e certifique-se de que o JUnit respeita rigorosamente a sequência 1, 2 e 3.

---

### 🟢 Nível 8: Embaralhando Testes com `MethodOrderer.Random`
**Cenário:** O arquiteto suspeita que alguns testes unitários estão passando por coincidência de ordem, dependendo de dados deixados por outros testes.
**Sua Tarefa:**
* Configure a anotação da classe para `@TestMethodOrder(MethodOrderer.Random.class)`.
* Execute a classe de testes múltiplas vezes.
* Observe na árvore de execução da IDE que a ordem dos métodos é alterada a cada rodada, comprovando a independência dos testes.

---

### 🟡 Nível 9: Ciclo de Vida por Classe com `@TestInstance(PER_CLASS)`
**Cenário:** Você precisa compartilhar o ID de um registro gerado no teste 1 com o teste 2, mas no ciclo padrão (`PER_METHOD`) a variável de instância volta a ser nula.
**Sua Tarefa:**
* No topo da classe, declare a anotação `@TestInstance(TestInstance.Lifecycle.PER_CLASS)`.
* Declare uma variável de instância `private String idGerado;`.
* No teste `@Order(1)`, atribua `this.idGerado = "ID-999";`.
* No teste `@Order(2)`, verifique com `assertEquals("ID-999", this.idGerado)`.
* Remova o modificador `static` de um método `@BeforeAll` dessa classe e comprove que ele compila e roda normalmente.

---

### 🟠 Nível 10: Integração Final (Fluxo Estatal de Pedido com Testes Parametrizados)
**Cenário:** Você precisa testar o fluxo de ciclo de vida completo de uma compra de e-commerce mantendo estado compartilhado e validando regras de desconto parametrizadas.
**Sua Tarefa:**
* Configure a classe com `@TestInstance(TestInstance.Lifecycle.PER_CLASS)` e `@TestMethodOrder(MethodOrderer.OrderAnnotation.class)`.
* Implemente:
  1. `@Test @Order(1)`: Cria o carrinho com itens e armazena o `totalBruto` em uma variável de instância.
  2. `@ParameterizedTest @Order(2) @CsvSource({"CUPOM10, 0.10", "CUPOM20, 0.20"})`: Valida a aplicação de diferentes cupons sobre o saldo acumulado.
  3. `@Test @Order(3)`: Finaliza a compra, esvazia o carrinho e assere que o total remanescente é zero.
* Execute a suíte e valide a perfeita harmonia entre testes parametrizados, ordenação explícita e ciclo de vida compartilhado.
