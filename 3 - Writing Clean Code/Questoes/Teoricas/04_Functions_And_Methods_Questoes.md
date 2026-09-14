# Questões Teoricas: Capítulo 04 — Functions & Methods (Funções e Métodos)

---

### Questão 1
Qual é a regra de ouro em relação à quantidade de parâmetros recebidos por uma função/método?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
O número de parâmetros deve ser **minimizado** ao máximo. O ideal é 0 parâmetros (excelente), seguido por 1 a 2 parâmetros (bom/aceitável). 3 parâmetros devem ser o limite evitado sempre que possível, e 4 ou mais parâmetros representam um design ruim e propenso a erros de ordenação.
</details>

---

### Questão 2
Quando uma função realmente necessita de 4 ou mais dados para operar, qual é a técnica recomendada de refatoração para tratar os parâmetros?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Agrupar os argumentos em um único Objeto ou Dicionário (Payload/Options Object). Dessa forma, a função passa a receber apenas 1 parâmetro, eliminando a dependência da ordem dos argumentos e melhorando a clareza na chamada através do acesso por chaves nomeadas.
</details>

---

### Questão 3
Por que os parâmetros de resto (Rest parameters `...args` ou `*args`) representam uma exceção válida à regra de limitação de argumentos?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Porque os argumentos são empacotados dinamicamente em um único Array interno e processados sob a mesma regra de negócio homogênea (ex: somar todos os números passados), mantendo a chamada simples e sem confusão de papéis individuais.
</details>

---

### Questão 4
O que são "Output Parameters" (Parâmetros de Saída), por que são considerados uma má prática e como eliminá-los?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Ocorrem quando uma função modifica silenciosamente o estado de um objeto que foi passado a ela como argumento de entrada. São ruins porque causam efeitos colaterais inesperados. Podem ser eliminados tornando o comportamento um método da própria classe do objeto (`user.addId()`) ou renomeando a função para deixar explícita a mutação.
</details>

---

### Questão 5
O princípio da Responsabilidade Única dita que funções devem "Fazer Apenas Uma Coisa" (Do One Thing). Como utilizar os Níveis de Abstração para determinar se uma função está fazendo apenas uma coisa?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Uma função faz apenas uma coisa quando todas as suas instruções internas pertencem ao **mesmo Nível de Abstração**, situado exatamente um degrau abaixo do nome da função. Se uma função de alto nível (regra de negócio) lida diretamente com APIs de baixo nível (manipulação direta de strings ou acesso nativo a disco), ela está misturando níveis e fazendo mais de uma coisa.
</details>

---

### Questão 6
O que determina o princípio DRY (Don't Repeat Yourself) e qual é o impacto de sua violação no sistema?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
O princípio DRY determina que cada pedaço de conhecimento ou regra de negócio deve ter uma representação única e inequívoca no sistema. A duplicação de lógica obriga alterações futuras a serem replicadas manualmente em múltiplos locais, aumentando o risco de bugs por esquecimento.
</details>

---

### Questão 7
O que caracteriza uma "Extração Inútil" de função e por que o exagero na divisão de funções prejudica o código?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Ocorre quando uma função é dividida sem ganho real de abstração, resultando em uma subfunção cujo nome é um sinônimo perfeito da função pai (ex: `buildUser()` dentro de `createUser()`). Esse exagero fragmenta o código desnecessariamente e dificulta o acompanhamento do fluxo de execução.
</details>

---

### Questão 8
Diferencie uma **Função Pura (Pure Function)** de uma função que gera **Efeitos Colaterais (Side Effects)**.

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
- **Função Pura:** Para os mesmos argumentos de entrada, sempre retorna o exato mesmo resultado e não altera nenhum estado fora de seu escopo.
- **Função com Side Effects:** Modifica o estado do mundo externo (ex: altera variáveis globais, escreve no banco de dados, grava arquivos ou imprime no console).
</details>

---

### Questão 9
Qual é o verdadeiro perigo associado aos Efeitos Colaterais (Side Effects) em métodos do sistema?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
O perigo não é a existência de efeitos colaterais (já que sistemas precisam deles para persistir dados ou comunicar-se), mas sim o fato de eles serem **inesperados**. Uma função cujo nome sugere apenas consulta/validação (ex: `isValidUser()`) jamais deve executar alterações de estado ou gravações ocultas.
</details>

---

### Questão 10
De que maneira a escrita de **Testes Unitários** atua como uma "prova de fogo" (indicador de saúde) para a qualidade das funções escritas?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Se uma função é difícil de ser testada em unidade — exigindo múltiplos *mocks* complexos ou testando dezenas de ramificações distintas —, isso indica que a função é grande demais, possui responsabilidades excessivas ou mistura múltiplos efeitos colaterais. Funções limpas, pequenas e puras são testadas de forma simples e rápida.
</details>
