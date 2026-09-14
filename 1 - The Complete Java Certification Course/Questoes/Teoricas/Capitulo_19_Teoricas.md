# Questões Teóricas - Capítulo 19 (Java Generics)

**1. A Era de Prata (Raw Types):** O que eram as famosas "Raw Types" (Coleções Cruas) antes do lançamento do Java 5, e qual era o grande perigo estrutural prático de usá-las em um sistema comercial?
> [!faq]- 👀 Ver Resposta
> Raw Types eram coleções, como o `ArrayList`, instanciadas sem a sintaxe de chaves angulares (`< >`). O perigo era que elas permitiam hospedar **qualquer tipo de objeto** (Strings, Inteiros e Booleanos misturados). Ao retirar o dado de lá, o programador tinha que fazer a conversão de tipo (*Casting* `(String) var`) manualmente às cegas. Se errasse a ordem, o programa explodia no rosto do usuário (Runtime Error).

**2. A Era de Ouro (Type Safety):** O que a expressão técnica "Type Safety" (Segurança de Tipo) significa e como os Generics do Java (`<T>`) blindaram o código contra explosões surpresa?
> [!faq]- 👀 Ver Resposta
> Significa garantir que o tipo exato de uma variável seja estritamente respeitado desde a codificação. Com Generics, se declararmos um `ArrayList<String>`, o próprio compilador/IDE do Java servirá de escudo. Ele bloqueará com linhas vermelhas e se recusará a rodar caso o programador tente colocar um Número lá dentro. O erro pula de tempo-de-execução (surpresa pro usuário) para tempo-de-compilação (barrado na cara do dev).

**3. O Molde Substituível:** Ao arquitetarmos uma classe genérica, como `public class Caixa<T>`, em que momento exato da vida da aplicação a letra fictícia `T` é substituída pelo tipo real e sólido de dado (como `Integer`)?
> [!faq]- 👀 Ver Resposta
> Essa substituição (Type Erasure / Instantiation) ocorre apenas no momento em que alguém em outra classe digita a invocação para dar vida ao objeto com o uso do 
ew`, por exemplo: `Caixa<Integer> c = new Caixa<>();`. Nesse momento, tudo dentro da classe molde passa a adotar `Integer` como regra.

**4. A Convenção Cifrada:** Qual é a convenção não-escrita, mas amplamente adotada por toda a comunidade Java, para a escolha e uso de letras na declaração dos "Type Parameters" das classes genéricas?
> [!faq]- 👀 Ver Resposta
> A comunidade adotou letras únicas e maiúsculas. O `T` é usado para o Tipo Genérico geral (Type). O `E` é usado para Elementos (em Listas/Sets). O `K` para a Key (Chave) e `V` para Value (Valor) em Mapas e Dicionários. O `N` para Números restritos.

**5. Assinatura do Método Genérico:** Caso o sistema exija um método estritamente genérico e que retorne aquele mesmo tipo misterioso, como deve ser a ordem visual e sintática das instruções na sua assinatura oficial?
> [!faq]- 👀 Ver Resposta
> A marcação do tipo fantasma (Ex: `<T>`) deve vir rigorosamente **antes** da instrução do tipo de Retorno final. Exemplo: `public static <T> T devolverMesmo(T elemento)`.

**6. O Bloqueio do Polimorfismo:** Por regra de herança no Java, sabemos que uma classe `Maca` é derivada (filha) da classe pai `Fruta`. Por que diabos o Java proibiria o polimorfismo clássico e consideraria estritamente PROIBIDO declarar que uma `List<Maca>` é igual/substitui uma `List<Fruta>`?
> [!faq]- 👀 Ver Resposta
> Para proteger a Type Safety! Se o Java permitisse que a variável base de `List<Fruta>` "adotasse" as listas filhas, outro pedaço do programa mal-intencionado poderia invocar a variável base, dar um comando de `.add(new Banana())` e inserir uma Banana pura dentro de uma lista que fisicamente era exclusiva para as filhas (Maçãs), corrompendo as maçãs sem a JVM perceber. Por isso ele bloqueia por completo.

**7. O Curinga Sem Regras (Unbounded):** Visando contornar o engessamento de proteção de tipos mencionado antes, a linguagem introduziu os Wildcards (Curingas - `?`). O que significa, para um parâmetro de método, exigir uma coleção assinada com `<?>`?
> [!faq]- 👀 Ver Resposta
> Significa "Qualquer Coisa, não importa a família". O método aceitará literalmente `List<Gatos>`, `List<Inteiros>` e `List<String>`. A limitação cruel disso é que, por não saber o que tem ali, o Java "lê" todos os elementos retirados de lá como sendo da classe raiz suprema do universo: a classe `Object`.

**8. O Teto e o Chão (Upper e Lower Bounds):** Na limitação dos "Wildcards", explique a regra sintática e semântica entre os operadores de Upper Bound (`? extends X`) e Lower Bound (`? super X`).
> [!faq]- 👀 Ver Resposta
> O Upper (`extends`) impõe que X é o "Teto" máximo hierárquico permitido. Só entram na lista a própria classe X e todos que nascerem "abaixo" dela (as suas classes filhas e netas). Já o Lower (`super`) define que X é o "Chão" base. Apenas a classe X e quem tiver gerado ela para "cima" (Pai, Avô e Object) são aceitos no método.

**9. O Que Passa na Porta? (Exercício Teórico):** Se o seu método Java possuir a declaração `public void verificarEquipe(List<? extends Programador> membros)`, a linguagem permitirá a injeção de uma lista de `WebDesigner` (filho da classe Designer)? E uma lista de `EngenheiroDeSoftware` (filho do Programador)?
> [!faq]- 👀 Ver Resposta
> Uma lista de `WebDesigner` seria ferozmente barrada por violação da árvore genealógica. Já uma lista de `EngenheiroDeSoftware` entraria de braços abertos, já que descende do "Teto" limite `Programador`.

**10. A Cegueira Pós-Teto:** Se usarmos um Wildcard focado no "Teto" máximo, digamos `List<? extends Funcionario>`, como o Java exibe/lida com os métodos dos objetos que tirarmos de lá de dentro através de um loop for-each?
> [!faq]- 👀 Ver Resposta
> O Java passa a tratar tudo como o elemento do "Teto" (a base segura). Mesmo se passarmos um Gerente ultra poderoso com métodos próprios, dentro desse loop o Java mascarará o objeto para parecer um mero `Funcionario` genérico, permitindo o uso exclusivo apenas de métodos que também existem no Funcionario (Teto), "cegando" os atributos que pertenciam apenas à classe-filha Gerente original.




