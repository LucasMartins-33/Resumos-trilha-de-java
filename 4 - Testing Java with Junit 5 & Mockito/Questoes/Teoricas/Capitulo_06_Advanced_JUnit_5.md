# Questões Teóricas - Capítulo 06: Tópicos Avançados do JUnit 5

Testes de fixação sobre testes parametrizados, fontes de dados, repetição de execução, ordenação de testes e ciclo de vida por classe.

---

### 1. O que são Testes Parametrizados (`@ParameterizedTest`) no JUnit 5 e qual problema eles resolvem na manutenção do código de teste?

<details>
<summary>👀 Ver Resposta</summary>

Testes parametrizados permitem executar o mesmo método de teste repetidas vezes utilizando diferentes conjuntos de dados de entrada como argumentos. Eles eliminam a duplicação de código e a proliferação de testes idênticos (*copy-paste*), permitindo testar diversas variações de valores válidos, inválidos e casos de contorno (boundary values) em uma única estrutura limpa e centralizada.
</details>

---

### 2. Em quais situações a anotação `@ValueSource` é adequada e qual é a sua principal limitação técnica?

<details>
<summary>👀 Ver Resposta</summary>

A anotação `@ValueSource` é adequada quando o método de teste precisa receber **apenas um único argumento** por execução a partir de um array simples de valores literais (como `strings`, `ints`, `doubles`, `longs`, `booleans` ou `classes`). Sua limitação técnica é a incapacidade de injetar múltiplos parâmetros simultaneamente (como um valor de entrada e um resultado esperado correspondente no mesmo teste).
</details>

---

### 3. Como funciona a anotação `@CsvSource` e como valores nulos e strings vazias devem ser representados nas suas linhas literais?

<details>
<summary>👀 Ver Resposta</summary>

A anotação `@CsvSource` aceita um array de strings formatadas no padrão CSV (valores separados por vírgula), onde cada item da linha é mapeado automaticamente para um parâmetro do método de teste com conversão de tipos automática do JUnit. Para representar uma **String vazia**, utiliza-se aspas simples contíguas (`''`). Para representar um valor nulo (**`null`**), omite-se qualquer conteúdo entre os delimitadores de vírgula (ou antes da vírgula).
</details>

---

### 4. Qual é o propósito da anotação `@CsvFileSource` e onde os arquivos de dados referenciados por ela devem ser armazenados no projeto?

<details>
<summary>👀 Ver Resposta</summary>

A anotação `@CsvFileSource` instrui o JUnit a ler a massa de dados de teste a partir de um arquivo CSV físico externo, desacoplando o código Java de grandes volumes de dados. Por convenção padrão em projetos Maven e Gradle, esses arquivos devem residir na pasta de recursos de teste (`src/test/resources/`) e ser referenciados através do atributo `resources = "/nome_do_arquivo.csv"`.
</details>

---

### 5. Por que a anotação `@MethodSource` é considerada a forma mais flexível de fornecer dados para testes parametrizados?

<details>
<summary>👀 Ver Resposta</summary>

Porque o `@MethodSource` consome dados gerados programmaticamente por um método em Java que retorna coleções ou fluxos do tipo `Stream<Arguments>`, `Collection<Arguments>` ou `Iterable<Arguments>`. Isso permite gerar dados dinâmicos, instanciar objetos complexos da aplicação, coleções aninhadas ou estruturas personalizadas que não podem ser expressas em texto puro como em anotações CSV.
</details>

---

### 6. Para que serve a anotação `@RepeatedTest` e em que contextos de validação de qualidade ela é mais utilizada?

<details>
<summary>👀 Ver Resposta</summary>

A anotação `@RepeatedTest(value = n)` força a execução consecutiva do mesmo teste por `n` vezes. Ela é utilizada principalmente para detectar e isolar testes instáveis (*flaky tests*), avaliar o comportamento de concorrência e *thread-safety* em sistemas multithread, ou verificar a estabilidade de rotinas de rede e conectividade que apresentam falhas intermitentes.
</details>

---

### 7. Como os parâmetros `RepetitionInfo` e `TestInfo` podem ser injetados em um método anotado com `@RepeatedTest`?

<details>
<summary>👀 Ver Resposta</summary>

O JUnit Jupiter utiliza seu mecanismo de injeção de parâmetros (`ParameterResolver`). Basta declarar `RepetitionInfo` ou `TestInfo` diretamente na lista de parâmetros da assinatura do método de teste. O framework resolve essas instâncias em tempo de execução, permitindo ao código do teste consultar em qual repetição atual está (`getCurrentRepetition()`) ou saber o total de iterações (`getTotalRepetitions()`).
</details>

---

### 8. Quais são os principais ordenadores fornecidos por `@TestMethodOrder` e por que a ordenação de testes deve ser evitada em testes unitários puros?

<details>
<summary>👀 Ver Resposta</summary>

Os principais ordenadores são:
* `MethodOrderer.Random.class`: Embaralha a ordem de execução a cada rodada.
* `MethodOrderer.MethodName.class`: Ordena pelo nome alfanumérico dos métodos.
* `MethodOrderer.OrderAnnotation.class`: Respeita o valor numérico da anotação `@Order(1)`, `@Order(2)` em cada método.
Em testes unitários puros, a ordenação deve ser evitada porque os testes precisam ser rigorosamente **independentes** (princípio *Independent* do F.I.R.S.T.). A necessidade de ordenar frequentemente mascara acoplamento e compartilhamento indevido de estado entre testes.
</details>

---

### 9. Qual a diferença fundamental entre o ciclo de vida padrão `TestInstance.Lifecycle.PER_METHOD` e o `TestInstance.Lifecycle.PER_CLASS`?

<details>
<summary>👀 Ver Resposta</summary>

* **`PER_METHOD` (Padrão):** O JUnit instancia um novo objeto da classe de teste para cada método `@Test` individual, zerando todas as variáveis de instância e isolando o estado por completo.
* **`PER_CLASS`:** O JUnit cria **apenas uma única instância** da classe de teste para executar todos os métodos de teste contidos nela. Com isso, os atributos e variáveis de instância mantêm seus valores entre um método de teste e o próximo, viabilizando fluxos de integração estatais (*stateful*).
</details>

---

### 10. Qual é a vantagem de sintaxe que o uso de `@TestInstance(Lifecycle.PER_CLASS)` traz para os métodos de configuração `@BeforeAll` e `@AfterAll`?

<details>
<summary>👀 Ver Resposta</summary>

Como existe apenas uma única instância da classe de teste viva durante toda a execução da suíte, os métodos anotados com `@BeforeAll` e `@AfterAll` **não precisam mais ser declarados como `static`**. Eles podem ser métodos comuns de instância, permitindo acessar diretamente atributos e variáveis de instância da classe de teste no momento da inicialização ou encerramento.
</details>
