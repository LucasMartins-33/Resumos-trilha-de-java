# Questões Teóricas - Capítulo 10: AOP (Aspect-Oriented Programming)

### Questão 1
Quais são os problemas de **Code Tangling** e **Code Scattering** na programação tradicional e como o AOP os resolve?

<details>
<summary>👀 Ver Resposta</summary>

* **Code Tangling**: Ocorre quando código de infraestrutura (logs, segurança, auditoria) fica misturado à regra de negócio principal dentro de um método.
* **Code Scattering**: Ocorre quando o mesmo código de infraestrutura precisa ser duplicado em dezenas ou centenas de classes em todo o sistema.

O AOP resolve esses problemas isolando essas preocupações transversais (*Cross-Cutting Concerns*) em módulos independentes chamados **Aspectos**, injetando-os dinamicamente sem alterar o código das regras de negócio.
</details>

---

### Questão 2
Defina os conceitos fundamentais de AOP: **Aspect**, **Advice**, **Pointcut**, **JoinPoint** e **Weaving**.

<details>
<summary>👀 Ver Resposta</summary>

* **Aspect**: Classe contendo código transversal de infraestrutura.
* **Advice**: Ação executada pelo aspecto e o momento em que é aplicada (`@Before`, `@AfterReturning`, etc.).
* **Pointcut**: Expressão que define as regras e locais onde o Advice será interceptado.
* **JoinPoint**: Ponto durante a execução do programa (ex: execução de um método) interceptado pelo AOP.
* **Weaving**: O processo de vincular aspectos aos objetos alvos (*Target Objects*).
</details>

---

### Questão 3
Quais são as principais diferenças entre o **Spring AOP** e o **AspectJ**?

<details>
<summary>👀 Ver Resposta</summary>

* **Spring AOP**: Mais simples, utiliza *Runtime Weaving* baseado no padrão Dynamic Proxy. Suporta apenas interceptação de execução de métodos em Beans gerenciados pelo Spring.
* **AspectJ**: Framework AOP completo, utiliza *Compile-Time*, *Post-Compile* ou *Load-Time Weaving*. Suporta interceptação de métodos, construtores e atributos em qualquer POJO Java.
</details>

---

### Questão 4
Dada a expressão de Pointcut `execution(* com.luv2code.aopdemo.dao.*.*(..))`, explique o significado de cada um de seus elementos.

<details>
<summary>👀 Ver Resposta</summary>

* `execution(...)`: Ponto de corte que intercepta a execução de métodos.
* O 1º `*`: Corresponde a qualquer tipo de retorno.
* `com.luv2code.aopdemo.dao`: Pacote-alvo da interceptação.
* O 2º `.*`: Qualquer classe pertencente a esse pacote.
* O 3º `.*`: Qualquer método pertencente a essa classe.
* `(..)`: Qualquer quantidade ou tipo de parâmetros (zero ou mais).
</details>

---

### Questão 5
Como controlar a ordem de execução de múltiplos aspectos que interceptam o mesmo método usando a anotação `@Order`?

<details>
<summary>👀 Ver Resposta</summary>

Deve-se colocar a anotação `@Order(N)` sobre a classe do Aspecto. **Menores números possuem maior precedência** (ex: `@Order(1)` executa antes de `@Order(2)`). Números negativos são permitidos.
</details>

---

### Questão 6
Qual é a função da anotação `@Pointcut` e como ela permite a reutilização e combinação de expressões de ponto de corte?

<details>
<summary>👀 Ver Resposta</summary>

A anotação `@Pointcut` permite declarar uma expressão de ponto de corte sobre um método de assinatura vazia (ex: `public void forDaoPackage() {}`). Esse método pode ser referenciado pelo nome em múltiplos Advices e combinado com operadores lógicos como `&&` (AND), `||` (OR) e `!` (NOT).
</details>

---

### Questão 7
Como acessar a assinatura do método interceptado e os argumentos passados a ele dentro de um Advice `@Before`?

<details>
<summary>👀 Ver Resposta</summary>

Adiciona-se o parâmetro `JoinPoint` no método do Advice:
* Assinatura: `MethodSignature signature = (MethodSignature) joinPoint.getSignature();`
* Argumentos: `Object[] args = joinPoint.getArgs();` (permite iterar sobre o array e inspecionar cada objeto recebido).
</details>

---

### Questão 8
Como funciona o Advice `@AfterReturning` e como ele permite inspecionar ou alterar os dados retornados por um método?

<details>
<summary>👀 Ver Resposta</summary>

O `@AfterReturning` executa somente após a conclusão bem-sucedida do método. Ele declara um atributo `returning = "result"` associado a um parâmetro no método do Advice. Como o objeto retornado é passado por referência (caso seja mutável, como uma lista), o aspecto pode alterar suas propriedades antes que ele seja entregue ao chamador.
</details>

---

### Questão 9
Qual é o comportamento do Advice `@AfterThrowing` em relação à propagação de exceções na aplicação?

<details>
<summary>👀 Ver Resposta</summary>

O `@AfterThrowing` permite capturar, inspecionar e logar a exceção lançada através do atributo `throwing = "theExc"`. No entanto, ele **não abafa nem consome a exceção**; ela continua seu fluxo normal de propagação para o chamador.
</details>

---

### Questão 10
O que torna o Advice `@Around` o mais poderoso do AOP e qual parâmetro obrigatório ele deve receber em sua assinatura?

<details>
<summary>👀 Ver Resposta</summary>

O `@Around` executa antes e depois do método-alvo, permitindo medir tempo de execução, alterar retornos e abafar ou tratar exceções. Ele deve receber obrigatoriamente o parâmetro **`ProceedingJoinPoint`** e invocar o método `proceed()` para dar prosseguimento à execução real do método interceptado.
</details>
