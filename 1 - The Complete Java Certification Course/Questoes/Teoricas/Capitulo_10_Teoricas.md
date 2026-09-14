# Questões Teóricas - Capítulo 10 (Strings, Nested Loops and Debugging)

**1. O Tipo String:** Por que a classe `String` não é considerada um tipo primitivo (como `int` ou `boolean`) em Java, e qual é a maior implicação prática disso?
> [!faq]- 👀 Ver Resposta
> Porque `String` é, na verdade, uma Classe (Objeto) na linguagem Java (note que começa com letra maiúscula). A implicação prática é que, por ser um objeto, ela vem equipada com uma imensa variedade de métodos utilitários embutidos (como `.length()`, `.toUpperCase()`, `.substring()`) que podemos chamar usando o "ponto", o que tipos primitivos não têm.

**2. A Matemática dos Índices:** Como funciona o sistema de indexação de uma String em Java? Se uma palavra tem tamanho 5 (como "TERRA"), quais são os números de seus índices de início e de fim?
> [!faq]- 👀 Ver Resposta
> O Java utiliza indexação "Zero-based" (começa do zero). Portanto, o primeiro caractere sempre estará no índice `0`, e o último caractere estará sempre na posição `length() - 1`. Para uma palavra de tamanho 5, os índices válidos vão de `0` até `4`.

**3. A Armadilha Mortal da Comparação:** Qual é o perigo e o erro conceitual de usar o operador lógico `==` para checar se duas Strings contêm o mesmo texto? Qual o jeito correto de compará-las?
> [!faq]- 👀 Ver Resposta
> Como Strings são objetos, o operador `==` não compara as letras em si, mas sim **o endereço de memória** onde os objetos estão morando. Se forem dois objetos diferentes criados com textos iguais, o `==` dará `false`. A forma correta e segura é usar o método `string1.equals(string2)`.

**4. A Lógica Exclusiva do Substring:** Explique o funcionamento do método `.substring(inicio, fim)`. Se executarmos `"JAVA".substring(0, 2)`, qual texto exatamente será devolvido e por quê?
> [!faq]- 👀 Ver Resposta
> O método corta e devolve um pedaço da String. A regra vital dele é: o índice de `inicio` é **inclusivo**, mas o índice de `fim` é **exclusivo** (o Java vai até ele, mas não inclui a letra daquela posição final). Portanto, `"JAVA".substring(0, 2)` retornará apenas `"JA"` (pega as letras dos índices 0 e 1, e para antes do 2).

**5. Ignorando a Caixa (Casing):** Em sistemas de login ou validação de e-mails, qual a diferença prática entre os métodos `.equals()` e `.equalsIgnoreCase()`?
> [!faq]- 👀 Ver Resposta
> O `.equals()` faz uma checagem restrita e *case-sensitive*, então "Lucas" será diferente de "lucas". O `.equalsIgnoreCase()` ignora totalmente letras maiúsculas e minúsculas, validando como `true` se as letras em si forem as mesmas, independentemente do "casing".

**6. Lidando com Resultados Inexistentes:** Se usarmos o método `.indexOf("Z")` para procurar a letra "Z" em uma String, mas essa letra não existir lá, qual é a resposta que o método devolve e o que esse número simboliza?
> [!faq]- 👀 Ver Resposta
> O Java retornará `-1`. Como todos os índices válidos no Java começam de `0` e vão para o infinito positivo, devolver um valor negativo é o padrão universal para sinalizar de forma segura que a busca falhou ou o item não foi encontrado.

**7. A Engenharia Reversa do For:** Descreva, em código ou com palavras, como deve ser configurada a estrutura tríplice de um loop `for` clássico para conseguir ler uma String **de trás para frente**.
> [!faq]- 👀 Ver Resposta
> 1) A variável de inicialização deve começar não do 0, mas do último índice: `int i = str.length() - 1;`. 2) A condição deve ser rodar enquanto for maior ou igual a zero: `i >= 0;`. 3) A atualização em vez de somar deve subtrair: `i--`.

**8. O Padrão dos Loops Aninhados:** O que é um "Nested Loop" (Loop aninhado) e como o fluxo de execução rítmico funciona entre o loop de fora (Outer) e o de dentro (Inner)?
> [!faq]- 👀 Ver Resposta
> É a prática de colocar um laço de repetição inteiro dentro do corpo de outro. O fluxo obedece a seguinte regra: para CADA passo ou iteração única que o loop externo dá, o loop interno tem que rodar seu ciclo completo (do começo ao fim). Se o externo roda 5 vezes e o interno 5, teremos 25 operações no centro.

**9. A Ameaça à Escalabilidade:** Por que os programadores e arquitetos de software evitam e temem tanto o uso indiscriminado de loops aninhados em bancos de dados grandes?
> [!faq]- 👀 Ver Resposta
> Por causa do efeito de crescimento exponencial (Complexidade $O(N^2)$). Se você tiver uma lista de 1.000 usuários e um loop aninhado de 1.000 perfis, o processador será forçado a fazer $1.000 \times 1.000 = 1.000.000$ (Um Milhão) de checagens. Com listas crescendo, loops aninhados podem congelar e derrubar a performance de um aplicativo inteiro.

**10. O Modo Raio-X (Debugger):** O que é um "Breakpoint" no contexto das ferramentas da IDE e como usar a função "Step Over" do Debugger é superior a espalhar vários `System.out.println("chegou aqui")` pelo código?
> [!faq]- 👀 Ver Resposta
> O *Breakpoint* é uma marca de "pare" que você coloca na linha do código. Ao rodar no modo Debug, o programa congela quando chega lá. O "Step Over" permite pular linha por linha lentamente, e a IDE exibe em tempo real as memórias e valores atuais de todas as variáveis no cantinho da tela. É infinitamente superior ao `println` porque você não suja seu código, pode enxergar dados ocultos que esqueceu de imprimir e inspeciona qualquer objeto profundamente.




