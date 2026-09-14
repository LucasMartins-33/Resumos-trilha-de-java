# Questões Teoricas: Capítulo 06 — Objects, Classes & Data Structures

---

### Questão 1
Qual é a diferença conceitual e arquitetural fundamental entre **Objetos Reais (Real Objects)** e **Estruturas de Dados / Contêineres (Data Structures / DTOs)**?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
- **Objetos Reais:** Escondem seu estado (atributos privados) e expõem comportamentos e abstrações públicas (métodos). Nós ordenamos o que o objeto deve fazer sem manipular seus dados internos diretamente.
- **Estruturas de Dados / DTOs:** Apenas armazenam e transportam dados. Não contêm regras de negócio ou lógicas complexas e expõem seus atributos de forma pública.
</details>

---

### Questão 2
Por que misturar Objetos Reais com Estruturas de Dados (ex: expor conexões privadas ou atributos internos de uma classe de domínio) prejudica o código?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Essa mistura gera alto acoplamento, violando o encapsulamento. Ela permite que código externo manipule diretamente os dados internos do objeto de formas não previstas, espalhando regras de negócio e validações por diversas partes do sistema.
</details>

---

### Questão 3
Como o **Polimorfismo** pode ser utilizado para refatorar classes que contêm múltiplos blocos `switch/case` ou `if/else` baseados em "tipos" de objetos?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Cria-se uma classe base ou interface comum com a assinatura do método genérico, e implementam-se subclasses especializadas para cada tipo. Cada subclasse encapsula sua lógica específica, eliminando completamente as checagens condicionais `if/switch`. O chamador apenas invoca o método da interface.
</details>

---

### Questão 4
O que estabelece o Princípio da Responsabilidade Única (SRP - Single Responsibility Principle) no nível de **Classes**?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
O SRP dita que uma classe deve ter **apenas uma razão para mudar**. Ela deve agrupar apenas comportamentos e dados que pertençam a uma mesma responsabilidade ou domínio lógico do sistema.
</details>

---

### Questão 5
O que é a **Coesão (Cohesion)** de uma classe e como identificamos se uma classe possui baixa coesão?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Coesão mede o grau de afinidade entre os métodos e os atributos de uma classe. Uma classe é altamente coesa quando a maioria dos seus métodos utiliza a maioria das suas propriedades internas. Se métodos utilizam apenas subconjuntos isolados de propriedades (ex: 3 métodos usam só `atributoA` e 3 usam só `atributoB`), a classe possui baixa coesão e deve ser dividida.
</details>

---

### Questão 6
O que dita a **Lei de Demeter** (Princípio do Menor Conhecimento) e qual o seu bordão principal ("Tell, Don't Ask")?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
A Lei de Demeter estabelece que um módulo ou classe não deve conhecer as entranhas ou estruturas profundas dos objetos com os quais interage (evitando o encadeamento profundo `a.b.c.d()`). O princípio "Tell, Don't Ask" orienta pedir diretamente para o objeto vizinho executar a ação desejada, em vez de solicitar os dados internos dele para processar por fora.
</details>

---

### Questão 7
Explique o significado do **Open-Closed Principle (OCP)** do SOLID.

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
As entidades de software (classes, módulos) devem estar **abertas para extensão, mas fechadas para modificação**. Deve ser possível adicionar novas funcionalidades ao sistema criando novas classes/extensões sem a necessidade de alterar o código-fonte original já existente e testado.
</details>

---

### Questão 8
O que determina o **Liskov Substitution Principle (LSP)** no contexto de herança de classes?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Classes derivadas (filhas) devem ser capazes de substituir suas classes base (pais) sem alterar a corretude do programa. Uma subclasse não deve sobrescrever comportamentos da classe pai de forma a violar o contrato original ou lançar exceções inesperadas para métodos herdados.
</details>

---

### Questão 9
Qual é a orientação trazida pelo **Interface Segregation Principle (ISP)**?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Clientes não devem ser forçados a depender de interfaces que não utilizam. É preferível criar múltiplas interfaces pequenas, específicas e focadas no consumidor do que uma única interface "gorda" e genérica que obriga classes a implementarem métodos irrelevantes para elas.
</details>

---

### Questão 10
Como o **Dependency Inversion Principle (DIP)** reduz o acoplamento entre módulos de alto nível e detalhes de baixo nível?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Módulos de alto nível (regras de negócio) não devem depender diretamente de módulos de baixo nível (implementações concretas como drivers de banco de dados ou bibliotecas de e-mail). Ambos devem depender de **abstrações** (Interfaces). As dependências concretas devem ser injetadas a partir de fora (Injeção de Dependência).
</details>
