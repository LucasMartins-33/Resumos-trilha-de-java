# Questões Teóricas - Capítulo 09: Cobertura de Código (Code Coverage)

Testes de fixação sobre métricas de cobertura, relatórios JaCoCo, complexidade ciclomática e interpretação de ramificações.

---

### 1. O que é Cobertura de Código (Code Coverage) e qual é o seu objetivo em projetos de software?

<details>
<summary>👀 Ver Resposta</summary>

Cobertura de Código é uma métrica de engenharia de software que afere a proporção percentual de classes, métodos, linhas ou instruções de bytecode que foram executadas durante a rodada de uma suíte de testes automatizados. Seu objetivo principal é identificar "pontos cegos" no sistema — áreas críticas de código de produção que ainda não possuem testes associados e representam risco potencial para a estabilidade da aplicação.
</details>

---

### 2. Por que atingir 100% de cobertura de código não significa que o software está totalmente livre de falhas (bug-free)?

<details>
<summary>👀 Ver Resposta</summary>

Porque a métrica de cobertura afere apenas se a linha foi executada pelo interpretador Java, e não se a lógica de negócio foi validada com rigor. Um teste pode invocar um método inteiro sem conter nenhuma asserção (`assertEquals`), gerando 100% de cobertura sem verificar nenhuma regra. Além disso, a métrica não detecta requisitos esquecidos, falhas de concorrência, problemas de integração ou respostas inesperadas para entradas não antecipadas pelo desenvolvedor.
</details>

---

### 3. Por que o mercado costuma adotar metas de cobertura entre 70% e 80% em vez de buscar obsessivamente os 100%?

<details>
<summary>👀 Ver Resposta</summary>

A busca pelos 100% de cobertura sofre com a lei dos rendimentos decrescentes: cobrir os últimos 15-20% do código (como getters, setters, blocos de tratamento de exceções catastróficas de I/O e configurações de frameworks) consome um esforço desproporcional para um ganho de segurança mínimo. Uma meta entre 70% e 80% concentra o esforço nos fluxos críticos de negócio e algoritmos complexos, oferecendo excelente retorno de investimento e alta confiança na manutenção.
</details>

---

### 4. Como funciona o recurso "Run with Coverage" no IntelliJ IDEA e o que indicam as marcações laterais no editor?

<details>
<summary>👀 Ver Resposta</summary>

O IntelliJ inicia a JVM acoplando um agente especial que rastreia em tempo real quais linhas de código foram tocadas durante o teste. No editor de código, a ferramenta pinta marcadores coloridos na calha lateral:
* **Barra Verde:** Indica que aquela linha específica foi percorrida por pelo menos um teste da suíte.
* **Barra Vermelha:** Indica que nenhum teste acionou aquela linha de código durante a execução.
</details>

---

### 5. No Maven, qual é o papel do `maven-surefire-report-plugin` e por que a propriedade `<testFailureIgnore>true</testFailureIgnore>` pode ser necessária?

<details>
<summary>👀 Ver Resposta</summary>

O `maven-surefire-report-plugin` transforma os arquivos XML gerados pelo Surefire em um documento HTML consolidado (`surefire-report.html`) exibindo o resumo geral de testes executados, tempos de resposta e falhas. A configuração `<testFailureIgnore>true</testFailureIgnore>` é necessária porque, por padrão, o Maven encerra o ciclo de vida imediatamente ao encontrar qualquer teste com falha, o que impediria o plugin de relatório de chegar a ser executado para gerar o arquivo HTML completo de auditoria.
</details>

---

### 6. Como o plugin **JaCoCo** (Java Code Coverage) opera no ciclo de vida do Maven para medir a cobertura de código?

<details>
<summary>👀 Ver Resposta</summary>

O JaCoCo utiliza duas fases fundamentais:
1. **`prepare-agent`:** Antes da execução dos testes, ele injeta um Java Agent na JVM que instrumenta o bytecode compilado em tempo de carregamento com contadores de execução.
2. **`report`:** Após a fase `test`, ele analisa os registros gerados pelo agente e compila os dados brutos em páginas HTML estruturadas e interativas no diretório `target/site/jacoco/index.html`.
</details>

---

### 7. O que representa a métrica "Missed Instructions" no relatório do JaCoCo?

<details>
<summary>👀 Ver Resposta</summary>

A métrica "Missed Instructions" avalia a quantidade de instruções elementares de bytecode Java (opcodes) que não foram executadas. Como uma única linha de código Java pode conter várias instruções de bytecode (como chamadas aninhadas ou atribuições compostas), a contagem de instruções provê uma medição técnica mais precisa e independente do formato de quebra de linhas do que a simples contagem de linhas físicas de texto.
</details>

---

### 8. O que é a métrica "Missed Branches" (Ramificações perdidas) e qual a sua relevância em estruturas condicionais?

<details>
<summary>👀 Ver Resposta</summary>

"Missed Branches" mede a quantidade de caminhos de desvio lógico (em estruturas como `if`, `else`, `switch` ou operadores ternários `?:`) que não foram exercitados. Sua relevância reside no fato de que uma linha com `if (a && b)` possui quatro caminhos possíveis de fluxo. Testar apenas a condição verdadeira executará a linha, mas deixará ramificações sem cobertura caso a ramificação alternativa de falsidade não seja testada.
</details>

---

### 9. O que é a métrica de Complexidade Ciclomática (Cyclomatic Complexity) exibida no JaCoCo?

<details>
<summary>👀 Ver Resposta</summary>

É uma métrica matemática criada por Thomas McCabe que quantifica o número de caminhos independentes lineares possíveis dentro de um bloco de código. Cada estrutura de decisão (`if`, `while`, `for`, `case`, `catch`) adiciona um novo caminho de ramificação. Uma complexidade ciclomática elevada sinaliza métodos densos, difíceis de ler, com alto risco de abrigar bugs e que exigirão um número substancialmente maior de testes unitários para atingir a cobertura completa de caminhos.
</details>

---

### 10. No relatório HTML do JaCoCo, qual é o significado das cores dos pequenos losangos (diamantes) exibidos ao lado das instruções condicionais?

<details>
<summary>👀 Ver Resposta</summary>

Os losangos sinalizam o nível de cobertura da ramificação lógica:
* 🟢 **Verde:** Todas as bifurcações lógicas daquela condicional (tanto o caminho `true` quanto o caminho `false`) foram exercitadas pelos testes.
* 🟡 **Amarelo:** O bloco condicional foi coberto apenas parcialmente (por exemplo, os testes executaram o fluxo verdadeiro, mas nenhuma bateria testou a condição falsa).
* 🔴 **Vermelho:** Nenhuma das bifurcações daquela decisão foi acionada pela suíte de testes.
</details>
