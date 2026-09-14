# Questões Práticas - Capítulo 05 (Fundamentos do JUnit 5 - Basics)

---

### 🟢 Nível 1: Dominando as Asserções Básicas
**Cenário:** Você tem uma classe `ContaBancaria` e precisa testar as condições fundamentais de saldo e titularidade.
**Sua Tarefa:**
* Crie a classe de teste `ContaBancariaTest`.
* Crie um método `@Test` e utilize as seguintes asserções da classe `Assertions`:
  * `assertEquals`: confira se o saldo inicial é `100.0`.
  * `assertNotEquals`: confira se o número da conta não é igual a `0`.
  * `assertTrue`: confira se a conta está ativa (`conta.isAtiva()`).
  * `assertFalse`: confira se a conta não está bloqueada (`conta.isBloqueada()`).
  * `assertNotNull`: garanta que o titular não é nulo.

---

### 🟡 Nível 2: Otimizando Performance com Lazy Assert Messages
**Cenário:** O seu teste precisa montar uma mensagem de erro complexa envolvendo o cálculo e a concatenação de múltiplos objetos pesados.
```java
// Código ineficiente que concatena a String SEMPRE, mesmo se o teste passar:
assertEquals(esperado, real, "Falha na conta " + conta.getNumero() + " para o cliente " + cliente.getNomeCompleto());
```
**Sua Tarefa:**
* Refatore a asserção acima utilizando uma expressão lambda vazia `() -> "sua mensagem"`.
* Explique por que passar um `Supplier<String>` melhora o desempenho da bateria de testes quando todas as asserções são bem-sucedidas.

---

### 🟠 Nível 3: Melhorando Relatórios com `@DisplayName`
**Cenário:** Os relatórios da sua esteira de CI estão cheios de nomes técnicos longos como `testSaque_QuandoSaldoForSuficiente_DeveSubtrairValorComSucesso()`.
**Sua Tarefa:**
* Adicione a anotação `@DisplayName` no nível da classe de teste: `"Testes das Operações de Conta Bancária"`.
* Adicione a anotação `@DisplayName` no método de saque: `"Saque: Deve decrementar o saldo com sucesso quando houver limite disponível"`.
* Execute o teste na IDE e observe a mudança visual na árvore de execução dos testes.

---

### 🔴 Nível 4: Isolando Instâncias com `@BeforeEach`
**Cenário:** Você percebeu que está instanciando `new Calculadora()` no início de todos os 8 métodos de teste da sua classe.
**Sua Tarefa:**
* Declare um atributo privado `private Calculadora calculadora;` na classe de teste.
* Crie um método anotado com `@BeforeEach` chamado `void setup()` que faça `this.calculadora = new Calculadora();`.
* Remova as instanciações manuais de dentro dos métodos `@Test` e confirme que cada teste passa a utilizar uma instância limpa e isolada.

---

### 🟣 Nível 5: Configuração de Infraestrutura Pesada com `@BeforeAll`
**Cenário:** A sua suíte de testes precisa inicializar uma conexão pesada (como um servidor de teste ou carregar um arquivo JSON de 50MB) que deve rodar **uma única vez** antes de qualquer teste.
**Sua Tarefa:**
* Crie um método com a anotação `@BeforeAll`.
* Declare o método obrigatoriamente como `static void setupGlobal()`.
* Imprima uma mensagem simulando a inicialização do recurso.
* Tente remover a palavra `static` e observe o erro de compilação ou exceção lançada pelo JUnit 5 informando sobre a obrigatoriedade de métodos estáticos no ciclo padrão.

---

### 🟤 Nível 6: Limpeza de Recursos com `@AfterEach` e `@AfterAll`
**Cenário:** Seus testes geram arquivos temporários no sistema de arquivos local que precisam ser apagados após cada execução para não consumir espaço em disco.
**Sua Tarefa:**
* Crie um método anotado com `@AfterEach` chamado `void tearDown()` para deletar os arquivos temporários criados pelo teste atual.
* Crie um método anotado com `@AfterAll` (estático) chamado `static void tearDownGlobal()` para fechar a conexão pesada aberta no Nível 5.
* Coloque `System.out.println` em cada um desses métodos e analise no console a sequência cronológica exata em que o JUnit 5 executa cada etapa do ciclo de vida.

---

### 🔵 Nível 7: Desabilitando Testes Quebrados com `@Disabled`
**Cenário:** Uma integração com a API de pagamentos está fora do ar para manutenção durante a sprint e quebrando o build.
**Sua Tarefa:**
* Anote o método de teste afetado com `@Disabled("Ignorado temporariamente: Gateway de pagamentos em manutenção até a Sprint 3")`.
* Execute a classe de testes na IDE.
* Verifique que o teste não roda, a barra permanece verde e o método é sinalizado com o ícone amarelo de "Pulado/Skipped", exibindo a justificativa.

---

### 🟢 Nível 8: Validando Exceções Esperadas com `assertThrows`
**Cenário:** A classe `Calculadora` possui o método `dividir(int dividendo, int divisor)`. Quando o divisor for `0`, o método deve disparar uma `ArithmeticException`.
**Sua Tarefa:**
* Escreva um método `@Test` chamado `testDivisaoPorZero()`.
* Utilize o `assertThrows` para validar o comportamento:
  ```java
  assertThrows(ArithmeticException.class, () -> {
      calculadora.dividir(10, 0);
  });
  ```
* Se o método não lançar exceção (por exemplo, retornar 0), o teste deve falhar.

---

### 🟡 Nível 9: Inspecionando a Mensagem Interna da Exceção
**Cenário:** Não basta saber que uma `IllegalArgumentException` foi disparada: a regra de negócio exige que a mensagem seja rigorosamente `"O valor do saque não pode ser negativo"`.
**Sua Tarefa:**
* Capture a exceção retornada pelo `assertThrows`:
  ```java
  IllegalArgumentException excecao = assertThrows(IllegalArgumentException.class, () -> {
      conta.sacar(-50.0);
  });
  ```
* Logo após, utilize `assertEquals("O valor do saque não pode ser negativo", excecao.getMessage())` para auditar o texto da mensagem.

---

### 🟠 Nível 10: Integração Final (Suíte Completa de Operações Financeiras)
**Cenário:** Você precisa entregar uma suíte de testes robusta e padronizada para uma classe `ServicoTransferencia`.
**Sua Tarefa:**
* A classe `ServicoTransferencia` possui o método `transferir(Conta origem, Conta destino, double valor)`:
  * Deve decrementar o saldo da conta de origem e incrementar a de destino.
  * Se o valor for menor ou igual a zero, deve lançar `IllegalArgumentException("Valor de transferência inválido")`.
  * Se a conta de origem não tiver saldo suficiente, deve lançar `SaldoInsuficienteException("Saldo insuficiente para transferência")`.
* Na classe de teste, utilize:
  * `@DisplayName` na classe e nos métodos.
  * `@BeforeEach` para criar contas zeradas com saldo controlado.
  * Testes para caminho feliz com `assertEquals` e Lazy Assert Messages.
  * Testes para ambos os cenários de erro com `assertThrows` auditando as mensagens.
  * Um teste anotado com `@Disabled` para uma funcionalidade futura de PIX agendado.
