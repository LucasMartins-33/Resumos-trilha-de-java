# Questões Teóricas - Capítulo 08: Mockito - Testando Código Java em Isolamento

Testes de fixação sobre isolamento de dependências, dublês de teste, anotações do Mockito, stubbing, verificação de interações e métodos void.

---

### 1. O que é um objeto Mock no contexto de testes unitários e qual é o seu principal objetivo?

> [!faq]- 👀 Ver Resposta
> Um Mock é um dublê de teste (*Test Double*) programável que substitui um componente real da aplicação (como um repositório de banco de dados ou um serviço de rede). Seu objetivo é simular o comportamento de dependências externas em memória e permitir o isolamento completo da classe que está sendo testada, garantindo testes rápidos, determinísticos e focados exclusivamente na lógica da unidade sob análise.

---

### 2. Qual é a responsabilidade da anotação `@ExtendWith(MockitoExtension.class)` em uma classe de teste JUnit 5?

> [!faq]- 👀 Ver Resposta
> A anotação `@ExtendWith(MockitoExtension.class)` conecta a extensão do Mockito ao ciclo de vida da JUnit Platform no JUnit 5. Ela é responsável por escanear a classe de teste antes da execução de cada método, inicializar todos os campos anotados com `@Mock`, e injetar essas instâncias falsificadas nos campos marcados com `@InjectMocks`, além de realizar validações de uso correto do framework.

---

### 3. Qual a diferença fundamental entre as anotações `@Mock` e `@InjectMocks`?

> [!faq]- 👀 Ver Resposta
> * **`@Mock`:** Cria uma instância simulada (falsa) de uma interface ou classe concreta. Todos os métodos desse objeto retornam valores padrão (como `null`, `0` ou `false`), a menos que sejam explicitamente instruídos via stubbing.
> * **`@InjectMocks`:** Instancia o objeto **real** da classe que queremos de fato testar e injeta automaticamente em seu construtor ou campos todos os dublês criados com `@Mock`.

---

### 4. O que significa o termo "Stubbing" no Mockito e qual estrutura de código é utilizada para implementá-lo?

> [!faq]- 👀 Ver Resposta
> *Stubbing* é o ato de pré-definir o comportamento de um mock, instruindo-o a responder com valores específicos ou exceções quando determinados métodos forem invocados. No Mockito, isso é configurado habitualmente na etapa Arrange utilizando a sintaxe fluente estática:
> `when(mockObject.metodo(argumentos)).thenReturn(valorDeRetorno);`

---

### 5. O que são Argument Matchers no Mockito (como `any()` ou `any(User.class)`) e qual regra deve ser observada ao combiná-los com valores literais?

> [!faq]- 👀 Ver Resposta
> Argument Matchers são predicados do Mockito que permitem configurar stubs ou verificações flexíveis, correspondendo a qualquer argumento que satisfaça um tipo ou padrão genérico. A regra estrita do Mockito é: **se você usar um matcher em um dos argumentos de um método, todos os outros argumentos da mesma chamada devem ser expressos através de matchers** (usando `eq("literal")` caso você precise de um valor exato).

---

### 6. Qual a diferença entre validação de estado (feita com `assertEquals`) e validação de comportamento (feita com `verify`)?

> [!faq]- 👀 Ver Resposta
> * **Validação de Estado:** Analisa o resultado final retornado por um método ou a alteração dos atributos de um objeto, comparando valores esperados com valores atuais.
> * **Validação de Comportamento:** Analisa as interações entre os componentes. Utilizando `verify()`, ela confirma se determinados métodos das dependências foram de fato invocados, com quais argumentos específicos e quantas vezes ocorreram durante a execução.

---

### 7. Como funcionam os modificadores de frequência de invocação do método `verify()` (como `times()`, `never()`, `atLeast()`)?

> [!faq]- 👀 Ver Resposta
> Eles delimitam a quantidade exata ou esperada de vezes que um método do mock deve ter sido chamado:
> * `verify(mock, times(1)).metodo(...)`: Exige que a chamada tenha ocorrido exatamente 1 vez.
> * `verify(mock, never()).metodo(...)`: Garante que o método jamais foi invocado durante o teste.
> * `verify(mock, atLeast(2)).metodo(...)`: Valida que o método foi chamado 2 ou mais vezes.
> * `verify(mock, atMostOnce()).metodo(...)`: Valida que ocorreu no máximo uma chamada (0 ou 1 vez).

---

### 8. Como o Mockito permite simular cenários de exceções em dependências com retorno (por exemplo, queda de conexão com banco de dados)?

> [!faq]- 👀 Ver Resposta
> Utiliza-se a instrução `.thenThrow()` na cadeia de stubbing:
> `when(repositorio.save(any())).thenThrow(new DataAccessException("Timeout"));`
> Quando a classe sob teste invocar o método `save`, o mock lançará a exceção programada, permitindo testar se a regra de negócio tratou, capturou ou encapsulou a falha corretamente (verificável com `assertThrows`).

---

### 9. Por que a sintaxe tradicional `when(mock.metodo()).thenReturn(...)` não compila em Java para métodos com retorno `void`?

> [!faq]- 👀 Ver Resposta
> Em Java, o tipo de retorno `void` não representa um valor ou objeto que possa ser passado como argumento de um método. Portanto, tentar passar `mock.metodoVoid()` como parâmetro para o método genérico `when(T value)` resulta em um erro de sintaxe do compilador Java, pois métodos `void` não produzem nada que possa ser consumido por outro método.

---

### 10. Qual é a família de comandos do Mockito para tratar métodos `void` e qual é a função de `doThrow()`, `doNothing()` e `doCallRealMethod()`?

> [!faq]- 👀 Ver Resposta
> Para métodos `void`, o Mockito inverte a sintaxe adotando o formato `do...().when(mock).metodoVoid()`.
> * **`doThrow(Excecao.class)`:** Força o método `void` do mock a disparar a exceção especificada.
> * **`doNothing()`:** Define que a chamada ao método `void` não terá qualquer efeito (comportamento padrão de mocks, útil para sobrescrever comportamentos globais de setup).
> * **`doCallRealMethod()`:** Ignora o stub e executa o código-fonte original contido dentro do método real do objeto.
