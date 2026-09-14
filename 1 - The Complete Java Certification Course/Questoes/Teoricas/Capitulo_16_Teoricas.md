# Questões Teóricas - Capítulo 16 (The Collections Framework)

**1. As Três Famílias:** Quais são as três grandes "famílias" (ou grupos principais) de estruturas de dados presentes no Java Collections Framework e qual o comportamento/foco principal de cada uma?
> [!faq]- 👀 Ver Resposta
> 1) **Lists (Listas)**: Mantêm a ordem de inserção e permitem elementos duplicados. 2) **Sets (Conjuntos)**: Não garantem ordem de inserção, mas bloqueiam veementemente elementos duplicados. 3) **Maps (Dicionários)**: Não usam índices, mas sim um sistema de pares de `Chave -> Valor`, onde a chave precisa ser obrigatoriamente única.

**2. Array vs Linked:** Qual a principal diferença de comportamento na memória e eficiência prática (velocidade) entre um `ArrayList` e uma `LinkedList`?
> [!faq]- 👀 Ver Resposta
> O `ArrayList` é um array com redimensionamento automático, sendo incrivelmente rápido para ler dados (ex: `get(50)` pula direto para a posição 50). A `LinkedList` é uma corrente de "vagões" (nós), sendo muito lenta para ler o item 50 (tem que percorrer do 1 ao 49), mas absurdamente rápida e eficiente para deletar ou inserir dados no meio da lista (basta soltar um engate e plugar o novo nó).

**3. O Ódio aos Primitivos:** O que acontece se tentarmos criar uma lista genérica de primitivos, como 
ew ArrayList<int>()`, e como o Java moderno contorna essa limitação?
> [!faq]- 👀 Ver Resposta
> O compilador lançará um erro, pois Coleções no Java só conseguem armazenar Objetos (Reference Types) na memória Heap. A solução é usar as **Classes Wrapper** correspondentes, como `Integer`, `Double`, ou `Boolean`.

**4. A Identidade dos Sets:** Qual é a regra inviolável de um `Set` e qual a principal diferença na saída de dados caso o programador decida usar um `LinkedHashSet` ao invés de um `HashSet` comum?
> [!faq]- 👀 Ver Resposta
> A regra de ouro do `Set` é nunca permitir dados duplicados. A diferença é que o `HashSet` é caótico e exibe os dados em uma ordem totalmente aleatória e imprevisível na tela, enquanto o `LinkedHashSet` respeita e devolve os dados exatamente na ordem cronológica em que foram inseridos no código.

**5. O Paradoxo do Cachorro Igual:** Por que somos obrigados a sobrescrever (dar Override) nos métodos `equals()` e `hashCode()` ao tentar impedir duplicatas num `HashSet` de objetos personalizados, como uma classe `Animal`?
> [!faq]- 👀 Ver Resposta
> Porque, por padrão, o Java compara o **endereço de memória** dos objetos. Dois animais diferentes com o nome "Rex" em dois comandos 
ew` distintos nascerão em endereços diferentes, então para o Java eles "não são duplicados". Ao sobrescrever o `equals()`, ensinamos a JVM que "se o nome for igual, considere-os o mesmo objeto e os bloqueie!".

**6. A Ordem Natural:** O método utilitário `Collections.sort()` consegue ordenar facilmente uma `List<String>`. O que a sua classe precisa ter para que ele consiga ordenar uma `List<Funcionario>`?
> [!faq]- 👀 Ver Resposta
> A classe `Funcionario` precisa assinar o contrato da interface `Comparable<Funcionario>` e, obrigatoriamente, implementar o método `compareTo(Funcionario outro)`, ensinando matematicamente ao Java se ele deve se basear na idade, nome ou salário para decidir quem vem primeiro na fila.

**7. A Anatomia do Dicionário:** Qual é a arquitetura lógica de armazenamento da estrutura `Map` em Java e por que ela não usa índices numéricos como o ArrayList?
> [!faq]- 👀 Ver Resposta
> O Map guarda os dados em pares indissociáveis (Key-Value / Chave-Valor). Ele não usa índices 0, 1, 2 porque a própria "Chave" (que pode ser uma String, um CPF, um E-mail) atua como o identificador único para "sacar" o valor (os dados da pessoa) instantaneamente do dicionário.

**8. O Confronto de Chaves:** O que acontece internamente em um `HashMap<String, String>` se você utilizar o método `.put("ChaveA", "Valor1")` e, na linha seguinte, mandar um `.put("ChaveA", "Valor2")`?
> [!faq]- 👀 Ver Resposta
> Como a regra máxima do Map dita que chaves não podem se repetir, o Java não dará erro. Em vez disso, ele achará o slot da "ChaveA" e **sobrescreverá (apagará e substituirá)** o "Valor1" pelo "Valor2" silenciosamente.

**9. Percorrendo o Labirinto:** Como podemos iterar (fazer um loop `for-each`) por todas as Chaves e Valores de um `Map` de maneira unificada e otimizada?
> [!faq]- 👀 Ver Resposta
> Usando a função `.entrySet()`. Podemos iterar declarando `for (Map.Entry<String, String> item : mapa.entrySet())`. Com isso, basta chamar `item.getKey()` para pegar a chave e `item.getValue()` para pegar a resposta atrelada a ela a cada iteração.

**10. O Filtro Destrutivo:** O que acontece com a `lista1` quando executamos o método `lista1.retainAll(lista2)`?
> [!faq]- 👀 Ver Resposta
> O método fará uma filtragem destrutiva. Ele vai apagar e varrer todos os itens da `lista1`, **mantendo apenas (retendo)** os itens que, por coincidência, também existirem dentro da `lista2`. É a famosa "Intersecção" de conjuntos.




