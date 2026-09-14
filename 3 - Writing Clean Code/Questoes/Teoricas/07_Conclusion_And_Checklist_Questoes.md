# Questões Teoricas: Capítulo 07 — Conclusão e Checklist Final do Clean Code

---

### Questão 1
Qual é a premissa central e filosófica que resume o objetivo final do Clean Code segundo a conclusão do curso?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Clean Code é sobre **escrever código para seres humanos, e não apenas para computadores**. Se o código é compreendido facilmente por outro desenvolvedor (ou por você mesmo no futuro), ele cumpriu seu papel de Código Limpo.
</details>

---

### Questão 2
Como as regras e diretrizes do Clean Code devem ser interpretadas pelos desenvolvedores no dia a dia de projeto?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Não como leis dogmáticas ou inquebráveis, mas como ferramentas e princípios guiados pelo bom senso e pela busca pragmática pela clareza e manutenibilidade.
</details>

---

### Questão 3
Quais são as principais diretrizes sobre a escolha das classes gramaticais (substantivos vs verbos) nos nomes de elementos de código?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
- **Substantivos:** Devem ser usados para nomear variáveis, propriedades de objetos e **Classes**.
- **Verbos:** Devem ser usados para nomear funções e métodos (representando ações).
</details>

---

### Questão 4
Por que a maioria dos comentários é vista pelo Clean Code como um "pedido de desculpas"?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Porque geralmente a presença de um comentário sinaliza que o desenvolvedor falhou em expressar a intenção daquele bloco de código através de boa nomenclatura, abstração adequada ou estrutura limpa, recorrendo ao comentário para explicar o que o código deveria ter tornado óbvio.
</details>

---

### Questão 5
Na Formatação Vertical, qual é a metáfora utilizada para descrever a ordem de leitura ideal das funções em um arquivo?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
A metáfora de uma **redação ou artigo de jornal**. As funções chamadoras (de mais alto nível) devem ficar no topo, e as funções chamadas (de nível imediatamente inferior) devem ser organizadas logo abaixo, permitindo uma leitura fluida de cima para baixo.
</details>

---

### Questão 6
Como deve ser tratada a lista de parâmetros de uma função no momento da revisão do código (Checklist)?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
A lista deve ser mantida o mais curta possível. Caso uma função exija 3 ou mais argumentos, eles devem ser agrupados em uma estrutura/objeto próprio de dados.
</details>

---

### Questão 7
Qual é a conduta esperada em relação a Efeitos Colaterais (Side Effects) ao revisar uma função?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Efeitos colaterais devem ser evitados. Caso sejam estritamente necessários para o propósito da função, o nome da função deve explicitar claramente essa alteração (ex: `saveUser()`), garantindo que a função não execute alterações ocultas incompatíveis com seu nome (ex: `isValidUser()` alterando o banco).
</details>

---

### Questão 8
O que a regra "Seja Positivo" orienta sobre o uso de variáveis e funções booleanas?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Orienta o uso de nomes e sentenças no afirmativo (`isReady`, `isEmpty`), pois expressões afirmativas reduzem o esforço mental exigido para processar condicionais complexas e evitam duplas negações (ex: `!hasNoElements`).
</details>

---

### Questão 9
Como os Princípios SOLID se conectam diretamente com o checklist de limpeza de Classes?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Garantem que as classes permaneçam pequenas e focadas em apenas uma responsabilidade (Single Responsibility), e permitam a inclusão de novas funcionalidades via extensão e polimorfismo em vez de modificação com acúmulo de `ifs` (Open-Closed Principle).
</details>

---

### Questão 10
Ao aplicar a técnica dos **Guards (Fail Fast)**, qual alteração visual imediata deve ocorrer na estrutura de uma função?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
A remoção imediata de recuos horizontais (aninhamento de `ifs`) e blocos `else` desnecessários, alinhando o caminho feliz (fluxo principal) da função na margem esquerda.
</details>
