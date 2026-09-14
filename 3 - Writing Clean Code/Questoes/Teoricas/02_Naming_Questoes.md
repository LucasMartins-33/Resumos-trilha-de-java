# Questões Teoricas: Capítulo 02 — Naming (Nomenclatura)

---

### Questão 1
Qual é a regra geral fundamental para a atribuição de nomes a variáveis, funções e classes no Clean Code?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
A regra fundamental é que os nomes devem ser **significativos** e revelar a intenção. Ao ler o nome de um elemento, deve ser possível compreender imediatamente o que ele armazena ou faz, sem a necessidade de inspecionar seu valor interno ou seu código de implementação.
</details>

---

### Questão 2
Explique a diferença de uso convencional entre os padrões de escrita `camelCase`, `snake_case` e `PascalCase`.

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
- **`camelCase`:** Primeira palavra em minúsculas e subsequentes em maiúsculas (ex: `userName`). Muito utilizado para variáveis, métodos e funções em JavaScript/TypeScript e Java.
- **`snake_case`:** Palavras em minúsculas separadas por caractere de sublinhado (ex: `user_name`). Padrão no Python para variáveis, funções e métodos.
- **`PascalCase`:** Todas as palavras iniciadas por maiúsculas (ex: `UserProfile`). Utilizado quase universalmente para nomes de **Classes** e Interfaces.
</details>

---

### Questão 3
Ao nomear variáveis que armazenam valores booleanos (verdadeiro/falso), qual técnica de estruturação de nome deve ser aplicada? Dê dois exemplos.

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Nomes de variáveis booleanas devem ser formulados como perguntas que possam ser respondidas com "sim" ou "não". 
Explicita-se o estado booleano utilizando prefixos como `is`, `has` ou `can`.
- Exemplo 1: `isActive`
- Exemplo 2: `hasPermission`
</details>

---

### Questão 4
Por que funções e métodos devem ser nomeados utilizando verbos, enquanto classes e variáveis utilizam substantivos?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Porque funções e métodos executam ações ou comportamentos (comandos), exigindo verbos para indicar o que está sendo realizado (ex: `saveUser()`). Variáveis armazenam dados e classes representam moldes de objetos ou conceitos estruturais (coisas), devendo ser representadas por substantivos (ex: `user`, `Customer`).
</details>

---

### Questão 5
Qual é a exceção à regra de utilizar verbos para nomear funções/métodos?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Funções ou métodos cujo propósito exclusivo é avaliar uma condição e retornar um valor booleano (verdadeiro/falso). Nesses casos, utiliza-se a mesma convenção dos predicados booleanos (ex: `emailIsValid()`, `isEmpty()`).
</details>

---

### Questão 6
O que é o erro de nomenclatura conhecido como "Desinformação" (Misleading Names)? Dê um exemplo prático.

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Ocorre quando um nome transmite uma ideia incorreta sobre a estrutura de dados, o tipo do valor ou o comportamento real do código. 
Exemplo: Nomear uma variável como `userList` quando a estrutura de dados subjacente é um Dicionário/Objeto (Mapa) e não um Array/Lista, ou nomear `allAccounts` uma coleção que já sofreu filtragem e contém apenas contas pagas.
</details>

---

### Questão 7
Por que nomes redundantes como `userWithNameAndAge` ou `coordX` dentro da classe `Point` devem ser evitados?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Porque o contexto ao redor do elemento já provê a informação necessária. Na classe `Point`, as propriedades `x` e `y` já são entendidas como coordenadas daquele ponto. Adicionar prefixos redundantes polui a leitura sem acrescentar clareza.
</details>

---

### Questão 8
O que postula a regra de "Consistência de Vocabulário" na escolha de verbos para funções?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Determina que se deve escolher uma palavra/verbo padrão para o mesmo conceito abstrato em todo o codebase. Se a palavra `get` foi definida para busca/recuperação de dados, não se deve alternar arbitrariamente entre `getUsers()`, `fetchProducts()` e `retrieveOrders()`. Um único termo deve ser adotado consistentemente.
</details>

---

### Questão 9
Em que cenários o uso de classes com sufixos como `Util` ou `Manager` é considerado aceitável?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
É aceitável apenas para **classes utilitárias** que agrupam coleções de funções/métodos estáticos puramente auxiliares e que não representam um objeto ou entidade do mundo real com estado próprio (ex: `DateUtil`).
</details>

---

### Questão 10
Como a transferência de uma função procedural solta para dentro de um método de classe pode melhorar a nomenclatura e o design do código?

<details>
<summary>👀 Ver Resposta</summary>

**Resposta:** 
Ao mover a função para dentro da classe do objeto que ela manipula, o próprio objeto fornece o contexto. Isso elimina a necessidade de passar o objeto como parâmetro adicional e reduz a necessidade de nomes longos e compostos na função (ex: transformar `print_blog_post_data(post)` na chamada limpa `post.print()`).
</details>
