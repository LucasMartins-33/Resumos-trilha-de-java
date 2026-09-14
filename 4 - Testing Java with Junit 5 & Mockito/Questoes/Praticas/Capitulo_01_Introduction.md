# Questões Práticas - Capítulo 01 (Introdução ao JUnit 5 e Testes Unitários)

---

### 🟢 Nível 1: O Primeiro Teste Unitário (Anotação `@Test`)
**Cenário:** Você acabou de criar uma classe simples `Calculadora` que possui um método `somar(int a, int b)`. Agora você precisa criar sua primeira verificação automatizada com JUnit 5.
**Sua Tarefa:**
* Crie a classe `CalculadoraTest`.
* Crie um método de teste com a anotação `@Test` do pacote `org.junit.jupiter.api.Test`.
* Deixe a visibilidade do método como padrão (*package-private*, sem a palavra `public`) e retorno `void`.
* Instancie a calculadora, chame `somar(2, 3)` e utilize `assertEquals(5, resultado)` importado estaticamente de `org.junit.jupiter.api.Assertions`.

---

### 🟡 Nível 2: Aplicando a Estrutura AAA (Arrange, Act, Assert)
**Cenário:** Um teste bagunçado mistura declaração de dados, execução de código e asserção na mesma linha. Para melhorar a legibilidade, sua equipe exige a separação visual do padrão AAA.
**Sua Tarefa:**
* Crie o teste `testSubtracao()`.
* Separe o corpo do método explicitamente com comentários nos três blocos:
  * `// Arrange`: Instancie a `Calculadora`, defina `int a = 10;`, `int b = 4;` e `int resultadoEsperado = 6;`.
  * `// Act`: Execute `int resultadoReal = calculadora.subtrair(a, b);`.
  * `// Assert`: Valide o resultado com `assertEquals(resultadoEsperado, resultadoReal)`.

---

### 🟠 Nível 3: Fornecendo Mensagens Customizadas de Falha
**Cenário:** Quando um teste falha em uma esteira de integração contínua (CI), mensagens genéricas dificultam a identificação rápida da falha.
**Sua Tarefa:**
* Escreva um método de teste para a multiplicação `testMultiplicacao()`.
* No `assertEquals`, passe o terceiro parâmetro contendo uma mensagem explicativa de erro (ex: `"O resultado da multiplicação de 5 por 4 deveria ser 20"`).
* Altere temporariamente o valor esperado para forçar o teste a falhar e observe a mensagem personalizada exibida no console da IDE.

---

### 🔴 Nível 4: Respeitando o Princípio "Independent" do F.I.R.S.T.
**Cenário:** Um desenvolvedor júnior declarou uma variável estática acumulativa dentro da classe de teste:
```java
class ContadorTest {
    static int contadorGlobal = 0;

    @Test
    void testeA() {
        contadorGlobal += 10;
        assertEquals(10, contadorGlobal);
    }

    @Test
    void testeB() {
        contadorGlobal += 5;
        assertEquals(5, contadorGlobal); // Falha se testeA rodar antes!
    }
}
```
**Sua Tarefa:**
* Identifique por que esse teste viola o princípio **Independent** do F.I.R.S.T.
* Refatore a classe: remova a variável estática compartilhada e garanta que cada teste inicialize seu próprio estado local e independente, rodando com sucesso em qualquer ordem.

---

### 🟣 Nível 5: Garantindo o Princípio "Self-Validating"
**Cenário:** Você encontrou um teste legado que não possui nenhuma asserção e apenas imprime valores no console:
```java
@Test
void testDivisaoLegado() {
    Calculadora calc = new Calculadora();
    int res = calc.dividir(10, 2);
    System.out.println("Resultado da divisão: " + res);
}
```
**Sua Tarefa:**
* Explique por que imprimir na tela viola a premissa de um teste automatizado autovalidável (*Self-validating*).
* Refatore o método eliminando o `System.out.println` e substituindo-o por uma validação assertiva com `assertEquals(5, res)`.

---

### 🟤 Nível 6: O Princípio "Thorough" (Cenários de Sucesso e Casos de Borda)
**Cenário:** A classe `Calculadora` possui o método `isNumeroPar(int numero)`. Testar apenas um número positivo par não é suficiente para garantir a robustez do algoritmo.
**Sua Tarefa:**
* Crie o método de teste `testIsNumeroPar_CenariosDiversos()`.
* Valide uma entrada par positiva (ex: 4 retornando `true` via `assertTrue`).
* Valide uma entrada ímpar positiva (ex: 7 retornando `false` via `assertFalse`).
* Valide o caso de borda com o número zero (`0` retornando `true`).
* Valide um número negativo par (ex: `-2` retornando `true`).

---

### 🔵 Nível 7: Identificando Acoplamento Rígido com o Operador `new`
**Cenário:** Você recebeu a seguinte classe de serviço para testar:
```java
public class PedidoService {
    private PedidoRepository repository = new PedidoRepository(); // Acoplamento rígido!

    public void salvarPedido(Pedido pedido) {
        repository.salvarNoBanco(pedido);
    }
}
```
**Sua Tarefa:**
* Por que instanciar o `PedidoRepository` diretamente com `new` impede que `PedidoService` seja testado em um teste unitário puro e veloz (*Fast*)?
* Refatore a classe `PedidoService` para utilizar **Injeção de Dependência via Construtor**: declare o `PedidoRepository` como atributo `final` e receba-o como parâmetro no construtor `public PedidoService(PedidoRepository repository)`.

---

### 🟢 Nível 8: Criando um Dublê de Teste Manual (Fake / Stub)
**Cenário:** Com o `PedidoService` desacoplado no Nível 7, você quer testar a lógica sem tocar no banco de dados real.
**Sua Tarefa:**
* Crie uma interface `PedidoRepository` com o método `boolean salvarNoBanco(Pedido pedido)`.
* Dentro da sua pasta de testes (`src/test/java`), crie uma classe dublê manual `FakePedidoRepository implements PedidoRepository` cujo método sempre retorne `true` em memória sem acessar o banco de dados.
* No seu teste unitário, instancie o `PedidoService` passando o seu `FakePedidoRepository` no construtor e valide o comportamento.

---

### 🟡 Nível 9: Mapeando Testes na Pirâmide de Testes
**Cenário:** O time de QA está debatendo quais tipos de testes devem ser escritos para uma nova funcionalidade de carrinho de compras de um e-commerce.
**Sua Tarefa:**
* Classifique cada um dos cenários abaixo no nível correspondente da Pirâmide de Testes (Unitário, Integração ou Ponta a Ponta / E2E):
  1. Testar o cálculo de 10% de desconto no método `calcularDesconto(cupom)` isolando o banco de dados.
  2. Subir um navegador automatizado com Selenium, clicar no botão "Comprar", preencher o formulário de cartão de crédito e validar a mensagem na tela.
  3. Salvar um carrinho real na base de dados PostgreSQL de homologação e verificar se as chaves estrangeiras de itens do pedido foram persistidas corretamente.

---

### 🟠 Nível 10: Integração Final (A Arquitetura Modular do JUnit 5)
**Cenário:** Você foi encarregado de preparar a base arquitetural para a equipe e precisa explicar como o JUnit 5 interage com a IDE e com o projeto.
**Sua Tarefa:**
* Crie uma classe de teste de exemplo `ArquiteturaJunit5Test`.
* Verifique em seu código os imports utilizados e responda em comentários de código estruturados:
  * De qual pacote do JUnit Jupiter vêm as anotações como `@Test`?
  * Qual é o papel da `JUnit Platform` quando você clica no botão "Play" da sua IDE (IntelliJ ou Eclipse)?
  * Se o projeto possuir testes antigos escritos em JUnit 4 (`org.junit.Test`), qual módulo do JUnit 5 deve estar presente no classpath para executá-los sem erros?
