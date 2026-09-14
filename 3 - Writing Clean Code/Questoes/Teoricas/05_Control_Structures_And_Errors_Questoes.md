# Questões Teoricas: Capítulo 05 — Control Structures & Errors (Estruturas de Controle e Erros)

---

### Questão 1
O que é a antipadrão de código conhecida como "Arrow Code" (Código em forma de flecha) e qual é a sua principal causa?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
"Arrow Code" é a deformação visual e estrutural provocada pelo aninhamento profundo de blocos de controle (`if/else` encadeados dentro de laços `for/while` e outros `ifs`). Isso empurra o código progressivamente para a direita, tornando a leitura confusa e difícil de acompanhar.
</details>

---

### Questão 2
O que é o padrão **Guards (Guardas / Fail Fast)** e de que maneira ele elimina o aninhamento profundo em funções?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
O padrão Guards consiste em inverter a verificação condicional no início da função para checar cenários de falha ou saída rápida. Em vez de envolver todo o corpo da função no bloco `if` positivo, valida-se o erro/exceção no topo e realiza-se um retorno antecipado (`return` ou `continue`), mantendo o fluxo principal linear e sem recuos.
</details>

---

### Questão 3
Por que a construção de expressões booleanas deve priorizar o "Fraseado Positivo" (Positive Phrasing)?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Porque o cérebro humano processa sentenças positivas de forma muito mais rápida e natural do que negações ou duplas negações. Funções e variáveis como `isEmpty()` ou `isReady()` são mais intuitivas de ler do que `hasNoElements()` ou `isNotInvalid()`.
</details>

---

### Questão 4
Quando uma condição dentro de uma instrução `if` torna-se complexa (ex: checar múltiplos status e propriedades em uma única linha), qual é a técnica correta de refatoração Clean Code?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Extrair a expressão condicional inteira para uma função auxiliar dedicada ou variável com nome semântico explicativo (ex: `if (isValidPayment(transaction))`).
</details>

---

### Questão 5
Por que a prática de retornar objetos de erro customizados (como `{ status: 500, error: true }`) em vez de lançar exceções reais (`throw new Error()`) é considerada uma má prática no Clean Code?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Porque obriga quem chama a função a tratar o erro como um valor normal através de verificações manuais com `if` adicionais, poluindo o fluxo de controle. Lançar exceções nativas aproveita o mecanismo da linguagem, permitindo que o erro borbulhe até o nível adequado de interceptação (`try/catch`).
</details>

---

### Questão 6
Como se aplica o princípio da Responsabilidade Única (Do One Thing) no contexto do tratamento de erros com blocos `try/catch`?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
O tratamento de erros (`try/catch`) deve ser a única responsabilidade da função em que ele habita. O bloco `try` deve apenas chamar a função que contém a execução da lógica real, sem misturar validações, laços de repetição ou regras de negócio dentro do mesmo bloco `try`.
</details>

---

### Questão 7
De que forma **Factory Functions** associadas a Dicionários/Mapas de funções auxiliam na eliminação de blocos extensos e repetitivos de `if / else if` baseados em tipos?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
A Factory Function mapeia os tipos para referências de funções dentro de um objeto/dicionário. Quem consome a fábrica obtém a referência adequada sem precisar executar lógicas imperativas de `if/else`, invocando o comportamento diretamente de forma polimórfica.
</details>

---

### Questão 8
Qual é o benefício do uso de Parâmetros Padrão (Default Parameters) na assinatura de funções no que tange às estruturas de controle?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Os parâmetros padrão eliminam a necessidade de escrever instruções `if` defensivas dentro do corpo da função para verificar se um argumento é `undefined` ou `null`, garantindo que o parâmetro sempre possuirá um valor inicial seguro.
</details>

---

### Questão 9
O que são "Números e Strings Mágicos" e por que eles devem ser substituídos por constantes ou Enums?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
São valores literais numéricos ou de texto inseridos diretamente no meio do código (ex: `if (status === 'CREDIT_CARD')` ou `if (userType === 2)`). Devem ser substituídos por constantes nomeadas ou Enums para evitar inconsistências por erros de digitação, centralizar alterações e explicitar o significado do valor.
</details>

---

### Questão 10
Em laços de repetição (`for` ou `while`), como as cláusulas `continue` funcionam como Guards?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Da mesma forma que o `return` funciona em funções: ao detectar uma condição inválida ou irrelevante no início da iteração, o `continue` interrompe o ciclo atual imediatamente e salta para o próximo elemento, evitando a necessidade de aninhar o código restante do loop dentro de um `if`.
</details>
