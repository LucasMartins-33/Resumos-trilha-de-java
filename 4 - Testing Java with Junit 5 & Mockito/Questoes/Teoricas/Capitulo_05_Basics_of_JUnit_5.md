# Questões Teóricas - Capítulo 05: Fundamentos do JUnit 5 (Basics)

Testes de fixação sobre asserções, mensagens preguiçosas, anotações de ciclo de vida e testes de exceções.

---

### 1. Qual é a convenção de ordem dos parâmetros no método `assertEquals` do JUnit 5 e qual a importância de respeitá-la?

<details>
<summary>👀 Ver Resposta</summary>

A assinatura padrão do `assertEquals` no JUnit 5 é:
`assertEquals(expected, actual, message)`.
A ordem exige primeiro o valor esperado (*expected*) e em seguida o valor real retornado pela execução (*actual*). Respeitar essa ordem é crucial para a legibilidade das mensagens de falha geradas pelo framework: se os parâmetros forem invertidos, o relatório de falha indicará incorretamente que o sistema "esperava o valor retornado mas obteve a constante esperada", gerando confusão durante o diagnóstico do erro.
</details>

---

### 2. O que são "Lazy Assert Messages" (mensagens preguiçosas com Lambdas) no JUnit 5 e qual é a sua principal vantagem em termos de performance?

<details>
<summary>👀 Ver Resposta</summary>

É o mecanismo no qual a mensagem descritiva de falha de uma asserção é fornecida por meio de uma expressão lambda `Supplier<String>` (por exemplo, `() -> "Falha com valor: " + obj.getId()`), em vez de uma String comum pré-concatenada. A vantagem é que a concatenação e a alocação de memória da mensagem só ocorrem se o teste de fato **falhar**. Se o teste passar com sucesso, o lambda nunca é avaliado, economizando processamento e memória em baterias extensas com milhares de testes.
</details>

---

### 3. Qual é o papel da anotação `@DisplayName` e por que ela é recomendada em suítes de testes profissionais?

<details>
<summary>👀 Ver Resposta</summary>

A anotação `@DisplayName` permite definir um nome descritivo, amigável e legível por humanos para uma classe ou método de teste, suportando espaços, caracteres especiais e emojis. Ela é recomendada porque nomes técnicos de métodos Java frequentemente se tornam longos e difíceis de interpretar em relatórios executivos ou pipelines de CI/CD. O `@DisplayName` transforma a saída dos testes em uma especificação funcional clara do que está sendo verificado.
</details>

---

### 4. Qual a diferença fundamental entre as anotações `@BeforeEach` e `@BeforeAll` no JUnit 5?

<details>
<summary>👀 Ver Resposta</summary>

* **`@BeforeEach`:** Executa antes de **cada** método de teste individual da classe. É utilizado para instanciar novos objetos e reinicializar o estado do cenário, garantindo isolamento entre os testes.
* **`@BeforeAll`:** Executa **uma única vez** antes de todos os métodos de teste da classe serem iniciados. É voltado para operações pesadas e compartilhadas de infraestrutura (como inicializar um banco de dados em memória ou carregar um arquivo de configuração volumoso).
</details>

---

### 5. Por que, por padrão, os métodos anotados com `@BeforeAll` e `@AfterAll` precisam ser declarados como `static` no JUnit 5?

<details>
<summary>👀 Ver Resposta</summary>

Por padrão, o ciclo de vida do JUnit 5 adota o modo `PER_METHOD`, criando uma nova instância da classe de teste para cada método `@Test` que executa. Como o método `@BeforeAll` precisa rodar antes mesmo da criação da primeira instância da classe de teste, ele deve pertencer à própria classe e não à instância, exigindo obrigatoriamente a palavra-chave `static`.
</details>

---

### 6. Para que serve a anotação `@AfterEach` e em quais situações ela é essencial?

<details>
<summary>👀 Ver Resposta</summary>

A anotação `@AfterEach` define um método de limpeza (*teardown*) que roda imediatamente após a finalização de cada método de teste, quer o teste tenha passado ou falhado. Ela é essencial para liberar recursos externos, deletar arquivos temporários criados em disco, fechar conexões de sockets ou redefinir variáveis de ambiente e propriedades do sistema, impedindo que o lixo de um teste contamine a execução do próximo.
</details>

---

### 7. Por que utilizar a anotação `@Disabled` é considerado uma prática muito melhor do que simplesmente comentar o código ou apagar a anotação `@Test` de um teste quebrado?

<details>
<summary>👀 Ver Resposta</summary>

Se você apagar a anotação `@Test` ou comentar o método, o teste deixará de existir para o executor do JUnit e se tornará código morto esquecido no repositório. Com a anotação `@Disabled("motivo")`, o JUnit continua ciente da existência do teste, computa-o nos relatórios como teste "Ignorado/Skipped" e exibe a justificativa cadastrada, alertando continuamente a equipe sobre uma pendência técnica que precisa ser resolvida.
</details>

---

### 8. Como funciona a asserção `assertThrows` e por que a ação sob teste (*Act*) é executada dentro de uma expressão lambda?

<details>
<summary>👀 Ver Resposta</summary>

O `assertThrows(Class<T> expectedType, Executable executable)` valida se a execução do bloco de código passado no lambda dispara uma exceção do tipo informado (ou de uma subclasse sua). O bloco de ação (*Act*) precisa ser encapsulado em um lambda `Executable` para que o JUnit consiga interceptar o fluxo em um bloco interno de `try-catch`. Se a exceção for lançada, a asserção é bem-sucedida; se nenhuma exceção for lançada, o JUnit falha o teste indicando que uma exceção era esperada.
</details>

---

### 9. O que o método `assertThrows` retorna e como esse retorno pode ser aproveitado para aprofundar a validação do teste?

<details>
<summary>👀 Ver Resposta</summary>

O `assertThrows` retorna a própria instância da exceção capturada (do tipo genérico `T`). O desenvolvedor pode armazenar esse retorno em uma variável e utilizar asserções adicionais (`assertEquals`, `assertTrue`) para inspecionar e validar detalhes internos do objeto da exceção, tais como a mensagem de erro exata (`exception.getMessage()`), códigos de erro customizados ou a causa raiz original (*cause*).
</details>

---

### 10. Qual é a diferença entre as asserções `assertTrue` / `assertFalse` e a asserção `assertEquals` para comparações booleanas?

<details>
<summary>👀 Ver Resposta</summary>

Embora `assertEquals(true, resultado)` e `assertTrue(resultado)` chequem a mesma condição lógica, `assertTrue(resultado, "mensagem")` é a forma idiomática recomendada, pois expressa a intenção de forma direta e concisa. Contudo, para comparar valores não booleanos (como objetos, números e Strings), deve-se sempre preferir `assertEquals(esperado, real)`, pois, em caso de falha, o JUnit exibe uma mensagem detalhada de comparação ("Expected: <X> but was: <Y>"), enquanto o `assertTrue` apenas reporta que o resultado booleano foi falso.
</details>
