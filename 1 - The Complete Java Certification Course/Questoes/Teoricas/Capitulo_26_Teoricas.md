# Questões Teóricas - Capítulo 26 (Lambda Expressions and the Streams API)

**1. A Morte da Classe Anônima:** O que é essencialmente uma Expressão Lambda e qual era a "monstruosidade" altamente verbosa e cansativa de escrever do antigo Java 7 que ela veio tentar substituir e encurtar drasticamente?
> [!faq]- 👀 Ver Resposta
> Lambda é, de forma simples, uma "Função Anônima" ou um bloco de código solto sem nome que você joga dentro de uma variável ou envia via parâmetro. Ela substituiu a trágica criação das "Classes Anônimas", onde o programador era forçado a usar de 5 a 6 linhas cheias de chaves, `@Override` e palavra 
ew` apenas para configurar uma função minúscula de soma de dois números.

**2. A Regra do Único Método:** Qual a regra fundamental e inquebrável da anatomia de uma Interface para que o compilador do Java a reconheça como uma "Functional Interface" legítima, permitindo a ancoragem de Expressões Lambdas ali?
> [!faq]- 👀 Ver Resposta
> A interface deve obrigatoriamente possuir apenas 1 (um) e somente 1 (um) **Método Abstrato** não-implementado. Se a interface abrigar mais do que isso, a flecha abstrata da função Lambda não saberá a qual dos dois métodos se conectar e "estourará" erro de sintaxe e compilação.

**3. O Selo de Garantia (@FunctionalInterface):** Apesar de não ser obrigatória, qual a enorme utilidade e importância prática de decorarmos nossa própria Interface Funcional com a Anotação `@FunctionalInterface` no topo?
> [!faq]- 👀 Ver Resposta
> Serve como um travamento de arquitetura. Se você colocar a anotação, você obriga a IDE e o Java a policiarem estritamente a Regra do Único Método (resposta da Q2). Desta forma, se no futuro um programador júnior sem aviso chegar lá e tentar colocar um "Segundo" método na interface sem querer, o código se recusará a compilar e evitará a destruição acidental da infraestrutura das suas chamadas de Lambdas ligadas àquilo.

**4. A Caixa de Condições (Predicate):** Com a explosão do Paradigma Funcional, a biblioteca interna `java.util.function` passou a vir pré-equipada com moldes padrão. Qual a função exata, e quais são as entradas e saídas da interface chamada `Predicate<T>`?
> [!faq]- 👀 Ver Resposta
> O Predicate é um validador universal focado em checar condições afirmativas/negativas e agir como uma roleta. Ele recebe pelo construtor um ÚNICO parâmetro de entrada flexível (T, como um carro, ou nome, idade) e é OBRIGADO invariavelmente a retornar um primitivo booleano (`boolean` de `true` ou `false`).

**5. A Fábrica de Modificações (Function):** Ao contrário da roleta lógica da anterior, a interface embutida `Function<T, R>` tem outra filosofia nativa de transformação. O que indicam as duas letras de tipo e para que a interface age primordialmente?
> [!faq]- 👀 Ver Resposta
> A interface age primariamente como um moinho e conversor dinâmico dos seus dados. A declaração da letra mística `T` engloba obrigatoriamente o Tipo de Entrada original que o método vai receber. A letra `R` encerra firmemente o tipo mágico final/alvo de Retorno que a função irá cuspir (Ex: recebe tipo Veículo `T` e transforma o chassi e cospe `String` `R` da cor modificada).

**6. A Magia da Torrente Fluida (Streams API):** O que é propriamente a inovação fenomenal chamada de "API de Streams" no Java 8, e qual o grave problema técnico/conceitual com o tradicional laço massivo `for/while` manual aplicado na varredura de coleções que a Stream tentou corrigir?
> [!faq]- 👀 Ver Resposta
> A Stream é o canal declarativo de pipeline de fluxos e transformações. O problema das interações velhas do modelo clássico e forçado (`for` iterador manual) era o volume gigante e complexo de regras estruturais necessárias onde você explicava exaustivamente o *COMO* deveria transacionar. A Stream adotou a filosofia focada do *O QUE* você quer que ele faça.

**7. O Meio x O Fim (Intermediate vs Terminal):** Descreva detalhadamente qual a extrema e profunda diferença conceitual no uso prático aplicado entre os fluxos classificados como "Intermediate Operations" (Operações Intermediárias) e as chamadas "Terminal Operations" (Operações Terminais) injetadas e engatadas num Stream?
> [!faq]- 👀 Ver Resposta
> As Intermediárias processam e distorcem, porem nunca acabam: elas funcionam devolvendo na saída mais e novas Streams sempre contínuas (permitindo assim o método `.encadeamento()`). Já as Terminais agem como ralo e represa. Ao usar o terminal, a Stream é drenada, encerrada violentamente para todos os sempre, e os fluidos decodificados e compilados são entregues de volta na forma física (variáveis e coleções puras encarnadas ou exibidos bruscamente).

**8. O Alterador de Estado (`.map()`):** A famosa operação e funil intermediário `.map()` (Mapeamento) processa que categoria específica de estrutura lambda por parâmetro obrigatório nas suas chamadas (Ex: Predicate, Callable, Function) para conseguir converter as correntes numéricas ou formatar os dados de cada linha do Duto da Stream perfeitamente?
> [!faq]- 👀 Ver Resposta
> A rotina do bloco do `.map()` solicita restritamente e unicamente uma Interface funcional do tipo explícito `Function<T, R>`. Porque a sua obrigação no túnel é devorar uma célula original exata e soltar para a esteira um produto transmutado ou moldado de volta para as próximas pontas da cadeia (A interface de entrada x retorno).

**9. A Ordem das Coisas (Filter antes do Sort):** Em questão bruta de processamento em memória nos servidores reais, por que o engenheiro/arquiteto recomenda rigorosamente ordenar a construção do seu Pipeline encaixando as rotinas de `.filter()` nas esteiras muito ANTES do peso morto da rotina encadeada do `.sorted()`?
> [!faq]- 👀 Ver Resposta
> Pelas estatísticas puras de processamento e ciclos da RAM. Se você rodar o `.sorted()` com 10.000 clientes, o motor gráfico de algoritmo do sistema exigirá recursos monstruosos na ordenação da pilha, para logo a seguir o seu `.filter()` expurgar e aniquilar sumariamente 8.000 perfis indesejados lixo (O Java os ordenou inteiramente de graça e jogou esforço brutal no lixo!). O `.filter()` corta primeiro e exaure os dados; o `.sorted()` agrupa apenas os sobreviventes minúsculos!

**10. Os Portões Finais:** Dê dois exemplos de portas ou Operações de Fluxo Terminais (Terminal Operations) universalmente famosas que são encarregadas de encerrar as Streams injetadas nos Pipelines modernos. E qual a finalidade básica e rápida delas?
> [!faq]- 👀 Ver Resposta
> Temos o brutal `.forEach()`, focado fundamentalmente em esgotar e anular o que sobrar invocando ações, modificações brutas e exibições diretas na tela; e o poderoso `.collect()` (e coletor `Collectors.toList()`), que funciona como funil inteligente, aprisionando, amontoando, recolhendo e solidificando de volta os fluxos soltos em blocos imutáveis puristas em formato nativo Array ou Lista limpa e funcional para alocar na Heap.




