# Questões Teoricas: Capítulo 03 — Code Structure, Comments & Formatting

---

### Questão 1
Qual é a premissa fundamental do Clean Code em relação ao uso de comentários no código-fonte?

> [!abstract]- Resposta
> **Resposta:** 
> Comentários devem ser evitados na maior parte do tempo. O código limpo deve ser autoexplicativo através de boa nomenclatura e estrutura clara. Na maioria dos casos, o comentário é uma tentativa de compensar ou explicar um código mal escrito.

---

### Questão 2
Por que comentários desnecessários podem se tornar perigosos ou enganosos com o passar do tempo em um projeto?

> [!abstract]- Resposta
> **Resposta:** 
> Porque o código evolui e passa por manutenções com frequência, mas os desenvolvedores muitas vezes se esquecem de atualizar os comentários associados. Com isso, o comentário deixa de corresponder à realidade do comportamento do sistema e passa a espalhar desinformação.

---

### Questão 3
Quais são os 4 principais exemplos de comentários ruins (que devem ser eliminados)?

> [!abstract]- Resposta
> **Resposta:** 
> 1. **Comentários Redundantes:** Repetem o que o próprio código já deixa óbvio.
> 2. **Divisores de Blocos:** "Desenhos" ou linhas divisórias no arquivo que tentam separar seções de código mal estruturado.
> 3. **Comentários Enganosos:** Afirmações que não condizem com a real execução do código.
> 4. **Código Comentado:** Blocos de código desativados mantidos por "segurança" (que deveriam ter sido deletados, confiando no sistema de controle de versão como o Git).

---

### Questão 4
Cite 4 situações em que o uso de comentários é considerado legítimo e encorajado (Bons Comentários).

> [!abstract]- Resposta
> **Resposta:** 
> 1. **Avisos Legais/Licenças:** Direitos autorais e licenças exigidas no topo do arquivo.
> 2. **Explicações Críticas de Lógica Complexa:** Explicar algoritmos matemáticos ou Expressões Regulares (RegEx) difíceis de compreender apenas pelo nome.
> 3. **Avisos Técnicos e Side-Effects:** Alertas sobre limitações de ambiente ou consequências colaterais não óbvias.
> 4. **Documentação de API Pública (JSDoc/Docstrings):** Orientar desenvolvedores externos ou integradores da biblioteca/API.

---

### Questão 5
O que postula a regra de "Formatação Vertical" e qual deve ser o tamanho ideal de um arquivo no Clean Code?

> [!abstract]- Resposta
> **Resposta:** 
> A Formatação Vertical organiza a disposição das linhas no arquivo. O arquivo deve ter poucas linhas de código, mantendo apenas um conceito ou uma classe principal por arquivo. As linhas em branco servem como delimitadores visuais entre seções lógicas distintas.

---

### Questão 6
Como deve ser ordenada a declaração de funções/métodos em um arquivo sob o conceito da Formatação Vertical?

> [!abstract]- Resposta
> **Resposta:** 
> As funções devem ser organizadas no estilo de uma redação ("de cima para baixo"). Uma função chamadora deve ser posicionada logo acima da função que ela invoca, permitindo que a leitura do arquivo ocorra de forma fluida sem a necessidade de saltar constantemente para cima e para baixo.

---

### Questão 7
Qual é o principal problema causado pela má "Formatação Horizontal" de um arquivo?

> [!abstract]- Resposta
> **Resposta:** 
> O surgimento da barra de rolagem horizontal. Linhas excessivamente longas forçam o desenvolvedor a rolar a tela lateralmente para entender a instrução completa, aumentando o esforço visual e o desgaste cognitivo.

---

### Questão 8
Como se deve refatorar uma linha de código longa e complexa para garantir boa Formatação Horizontal?

> [!abstract]- Resposta
> **Resposta:** 
> Quebrando as instruções encadeadas ou condicionais extensas em múltiplas linhas através do uso de variáveis intermediárias temporárias com nomes semânticos que expliquem cada subetapa.

---

### Questão 9
O que são linhas em branco na Formatação Vertical e como devem ser utilizadas?

> [!abstract]- Resposta
> **Resposta:** 
> As linhas em branco funcionam como parágrafos ou vírgulas visuais. Elas devem ser usadas para separar blocos conceituais distintos (ex: entre declaração de atributos, construtor e métodos), mas não devem ser usadas em excesso dentro de um mesmo bloco de código correlacionado.

---

### Questão 10
Como o comportamento de *Hoisting* no JavaScript afeta a Formatação Vertical quando comparado ao funcionamento do Python?

> [!abstract]- Resposta
> **Resposta:** 
> No JavaScript, o *hoisting* permite que funções declaradas com a palavra-chave `function` sejam chamadas antes de sua linha de declaração física, facilitando a formatação top-down (chamadora acima, chamada abaixo). No Python, as funções precisam ser declaradas antes de serem invocadas no fluxo do script, o que impõe restrições estritas à ordem de leitura das funções.
