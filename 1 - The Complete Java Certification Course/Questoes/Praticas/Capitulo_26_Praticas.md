# Questões Práticas - Capítulo 26 (Lambda Expressions and the Streams API)

🟢 Nível 1: Desenhando a Interface (Functional Interface)
Cenário: Vamos criar um componente que diz se um carro obedece às regulamentações antigas de emissão da Europa.
Sua Tarefa: 
* Desenvolva o contrato de arquivo limpo `public interface ValidadorVeicular`.
* Coloque a anotação moderna que impede a interface de inchar acidentalmente.
* Desenvolva um (e apenas um) método abstrato nativo: `boolean validarEmissao(Carro c);`.

🟡 Nível 2: O Desapego ao Override (Injeção Lambda)
Cenário: Vamos dar vida à nossa ferramenta do Nível 1 encurtando e abolindo toda e qualquer menção a classes anônimas pesadas.
Sua Tarefa:
* Na classe mestre do projeto, dentro de um escopo normal, instancie uma variável apontando para a interface `ValidadorVeicular verificador =`.
* Pule o instanciamento brutal, digite as chaves angulares e a flecha mágica das Expressões Lambdas: `(c) -> { return c.anoEmissao >= 2012; };`. 
* Agora as engrenagens já podem chamar no Java nativo a variável e interagir usando `verificador.validarEmissao(novoCarro);`.

🟠 Nível 3: O Predicate Embutido Nativo
Cenário: Seu arquiteto jogou o `ValidadorVeicular` fora dizendo que reinventar a roda era inútil, já que o Java 8 entregou ferramentas gratuitas.
Sua Tarefa:
* Importe `java.util.function.Predicate`.
* Declare estritamente um Predicate focado puramente e instanciado em objetos Carro. O Lambda encarregado de validar o ano passará a se chamar dinamicamente usando a API interna: `Predicate<Carro> validadorOficial = c -> c.anoEmissao >= 2012;`.
* *(Observe que quando o Lambda tem apenas uma instrução curtíssima sem Ifs massivos, somos livres para amputar e arrancar os parênteses, as chaves de blocos `{}` e, também miraculosamente, a palavra-chave enfadonha `return`, mantendo o código orgânico)*.

🔴 Nível 4: O Modificador Molecular (Function)
Cenário: Recebemos IDs sujos de registro do Detran no formato `Carro:3842-12` e devemos transmutá-los fisicamente devolvendo e expelindo os pedaços em formato numeral primário para o Database usar nas chaves!
Sua Tarefa:
* Adicione da engrenagem do utils o método matriz: `Function<String, Integer> formatador =`.
* O Lambda a seguir absorve o chassi, invoca a ferramenta do capítulo passado das strings (`.substring(6)`) expurgando o lado cego "Carro:", arranca o traço no processador (`.replace("-", "")`) encadernando tudo, e cospe de volta injetando nas entranhas de um `Integer.parseInt(limpo)`. Tente elaborar em poucas linhas ou desenhar mentalmente.

🟣 Nível 5: Abrindo a Represa (Source Stream)
Cenário: Temos os ingredientes, abriremos o Duto Supremo de processamentos dinâmicos!
Código de Partida:
```java
List<String> placasTransito = Arrays.asList("AAA123", "XPT990", "AAV889", "ZZZ111");
// Crie o pipeline!
```
Sua Tarefa:
* Simplesmente abra e ative o motor orgânico de canais chamando a ignição do Duto na lista e injetando: `placasTransito.stream()`.

🟤 Nível 6: A Filtragem Massiva e Feroz (`.filter`)
Cenário: O rastreador da blitz pede para caçarmos unicamente placas que estritamente comecem e ativem a zona restrita da letra "A".
Sua Tarefa:
* Encadeie, colando um "ponto", logo ao término lógico do comando acima, o núcleo da operação Intermediária que fará isso.
* Injetando o método brutal: `.filter( p -> p.startsWith("A") )`. A Stream irá bloquear os carros intrusos e devolver um rio restrito livre das poluições.

🔵 Nível 7: Polindo a Matéria Pura (`.map`)
Cenário: As placas que sobreviveram ao `.filter` do nível anterior e continuaram escorrendo na tubulação estavam bagunçadas, os sensores querem formatá-las engastando o ícone do estado de onde o carro vazou no cabeçalho.
Sua Tarefa:
* Na linha abaixo do `.filter(...)`, jogue um Enter. Invoque de maneira idêntica a função mutante intermediária e alteradora chamada de `.map( p -> "SP-" + p )`.
* Se o sobrevivente da onda anterior entrava cru como `AAA123`, a partir de agora esse mesmo corpo continuará escorrendo mutado sendo o material encadernado puro e absoluto `SP-AAA123`.

🟢 Nível 8: Fechando os Dutos (Terminal)
Cenário: O processador da Stream parou o tempo. Intermediárias sem Terminais ficam em coma ("Lazy Execution") estaticamente.
Sua Tarefa:
* Encerre firmemente e aniquile a esteira mágica final anexando o bloco do ralo: `.forEach(resultadoFinal -> System.out.println(resultadoFinal));`. O programa processará todos os blocos ativando as roldanas da API num milissegundo de operação ininterrupta e unificada sem o uso sujo do boilerplate antigo for/if manual!

🟡 Nível 9: Refatorando (Desafio de Modernização)
Cenário: Converta a mentalidade antiga na tecnologia nativa unificada da versão 8+.
Código de Partida:
```java
List<Integer> notas = Arrays.asList(4, 8, 2, 9, 10);
int somatorio = 0;
for(Integer n : notas) {
    if(n >= 7) {
        somatorio += n;
    }
}
```
Sua Tarefa:
* Remova o for cru, o If bruto, a contabilidade desnecessária de `int somatorio=0` em linha avulsa. Drene a coleção pela Stream, use um funil filtro (`.filter(n -> n >= 7)`), e acione a operação estritamente matemática embutida de soma automática invocando um reducionismo na terminal combinando o Duto das Primitivas: `.mapToInt(n -> n).sum();`.
* Acople perfeitamente o dreno retornando em inteiros absolutos na ponta e mate o desafio.

🟠 Nível 10: O Mestre do Duto (O Grande Pipeline)
Cenário: Mostre o potencial destrutivo total e elegante unificado das Coleções em poucas fatias!
Sua Tarefa (Apenas teórico para entender e recitar as fases do canal):
1. Injetamos Listas Sujas -> `clientes.stream()`
2. Filtramos Ativos Finais -> `.filter(c -> c.isAtivo())`
3. Convertemos Entidades em Nomes (A extração de núcleo leve Function) -> `.map(c -> c.getNome())`
4. Re-alinhamos Otimamente Ordem Alfabética -> `.sorted()`
5. Solidificamos em Banco Estático Nativo -> `.collect(Collectors.toList());`
* Em incríveis (ou míseros) apenas 5 furos lógicos diretos e em cadeia, a equipe acabou de esmagar, obliterar e resolver um fluxo imenso de algoritmos pesados de arquitetura for/if manual de mais de 25 linhas, e ainda aumentou expressivamente a visibilidade limpa das Regras de Negócio e o ganho extremo da Manutenção e paralelismo!
