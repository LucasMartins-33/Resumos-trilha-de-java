# Questões Teóricas: Capítulo 4 - Princípios SOLID

1. **Single Responsibility Principle (SRP):** Como o conceito de "Um único motivo para mudar" ajuda a definir os limites da responsabilidade de uma classe?
<details>
<summary>👀 Ver Resposta</summary>

O SRP estabelece que uma classe deve ter apenas uma única responsabilidade no sistema, o que significa que ela deve responder a apenas um "ator" ou contexto de negócio. O critério de "um único motivo para mudar" ajuda a identificar limites claros: se uma mudança nas regras fiscais E uma mudança na formatação do layout de impressão exigem alterações na mesma classe `Fatura`, essa classe tem mais de um motivo para mudar, violando o princípio. Separar essas responsabilidades em classes distintas reduz impactos colaterais e simplifica a manutenção.
</details>

2. **Violação do SRP:** Quais são os sinais clássicos e os problemas práticos enfrentados por uma classe que viola o SRP (as chamadas *God Classes*)?
<details>
<summary>👀 Ver Resposta</summary>

Os sinais clássicos incluem: arquivos gigantescos com centenas ou milhares de linhas, dezenas de dependências injetadas ou importadas, métodos com nomes desconexos e uma mistura desordenada de regras de negócio, persistência de banco de dados e formatação visual. Os problemas práticos são: testes unitários extremamente complexos de montar, alto risco de regressão (uma alteração em um recurso quebra outro não relacionado), dificuldades para paralelizar tarefas na equipe e conflitos constantes no controle de versão (Git merge conflicts).
</details>

3. **Open/Closed Principle (OCP):** Explique a frase: "Entidades de software devem ser abertas para extensão, mas fechadas para modificação".
<details>
<summary>👀 Ver Resposta</summary>

Significa que quando novos requisitos, regras de negócio ou funcionalidades forem introduzidos no sistema, devemos ser capazes de expandir o comportamento existente criando **novo código** (extensão), em vez de alterar o código-fonte original já testado e em produção (fechado para modificação). Isso protege a estabilidade do sistema contra efeitos colaterais e bugs regressivos.
</details>

4. **Polimorfismo no OCP:** Como a adoção de interfaces e polimorfismo facilita o cumprimento do Princípio Aberto/Fechado?
<details>
<summary>👀 Ver Resposta</summary>

Ao programar voltado para uma **interface** em vez de classes concretas, o fluxo principal do sistema interage com uma abstração (ex: `CalculadoraDesconto`). Quando um novo tipo de desconto precisar ser adicionado (ex: `DescontoBlackFriday`), basta criar uma nova classe que implemente a interface e injetá-la no sistema. A classe consumidora permanece 100% intacta, eliminando longas estruturas condicionais `if/else` ou `switch/case` e cumprindo perfeitamente o OCP.
</details>

5. **Liskov Substitution Principle (LSP):** O que esse princípio exige sobre a relação e o comportamento esperado entre uma classe derivada e sua classe base?
<details>
<summary>👀 Ver Resposta</summary>

O LSP, formulado por Barbara Liskov, exige que qualquer classe derivada (subclasse) possa substituir perfeitamente sua classe base (superclasse) sem quebrar o comportamento, os contratos ou a corretude do programa. Isso significa que as subclasses devem respeitar os mesmos contratos, não podem fortalecer as pré-condições (exigir mais do que o pai exigia), nem enfraquecer as pós-condições (garantir menos do que o pai garantia), e nunca devem lançar exceções inesperadas para métodos herdados.
</details>

6. **O Paradoxo do Quadrado e Retângulo:** Por que fazer a classe `Quadrado` herdar de `Retangulo` matematicamente faz sentido, mas na programação orientada a objetos é uma clássica violação do LSP?
<details>
<summary>👀 Ver Resposta</summary>

Na matemática, todo quadrado é um retângulo especial. No entanto, em POO com estado mutável, a classe `Retangulo` possui os métodos `setLargura(l)` e `setAltura(a)` com o contrato implícito de que alterar a largura não modifica a altura. Para manter suas características geométricas, um `Quadrado` sobrescreveria esses métodos alterando ambos os lados simultaneamente. Um cliente que espera um `Retangulo` e define largura 5 e altura 4 terá uma área inesperada de 16 ou 25 em vez de 20, quebrando a expectativa e violando frontalmente o LSP.
</details>

7. **Interface Segregation Principle (ISP):** O que são "interfaces gordas" (fat interfaces) e por que o ISP sugere que devemos dividi-las em interfaces mais granulares?
<details>
<summary>👀 Ver Resposta</summary>

"Interfaces gordas" são aquelas que acumulam métodos para muitas operações distintas, obrigando as classes implementadoras a dependerem de métodos que elas não precisam nem utilizam. O ISP prega que "nenhum cliente deve ser forçado a depender de métodos que não usa". Dividir essas interfaces em contratos menores, focados e coesos (interfaces granulares) permite que cada classe implemente apenas o que realmente lhe diz respeito, mantendo o design limpo e desacoplado.
</details>

8. **Violação do ISP:** O que acontece quando uma classe é forçada a implementar uma interface com métodos que ela não utiliza? Qual erro comum costumamos ver nesses métodos?
<details>
<summary>👀 Ver Resposta</summary>

A classe é contaminada com métodos vazios ou métodos que lançam exceções de recusa, como `throw new UnsupportedOperationException("Método não suportado")`. Isso polui a API da classe, quebra o polimorfismo e viola o princípio de substituição de Liskov (LSP), pois um consumidor que chamar esse método com base na interface sofrerá uma falha em tempo de execução inesperada.
</details>

9. **Dependency Inversion Principle (DIP):** Explique a regra central do DIP sobre módulos de alto nível e módulos de baixo nível.
<details>
<summary>👀 Ver Resposta</summary>

A regra central do DIP possui duas cláusulas essenciais:
1. Módulos de alto nível (que contêm as regras de negócio centrais) não devem depender de módulos de baixo nível (detalhes técnicos como bancos de dados, envio de e-mails, frameworks ou interfaces gráficas). Ambos devem depender de **abstrações** (interfaces ou classes abstratas).
2. As abstrações não devem depender de detalhes de implementação; os detalhes é que devem depender das abstrações.
</details>

10. **Abstrações no DIP:** Por que abstrações não devem depender de detalhes, mas os detalhes devem depender de abstrações? Dê um exemplo prático.
<details>
<summary>👀 Ver Resposta</summary>

Porque as regras de negócio são o ativo mais valioso e estável do software, enquanto tecnologias, drivers de banco de dados e APIs externas são detalhes transitórios que mudam com frequência. Se a regra de negócio depender diretamente de um detalhe concreto (ex: `MySQLDatabase`), ela se torna refém dessa tecnologia. 
**Exemplo prático:** Em vez de a classe `ProcessadorDePagamento` instanciar diretamente `MySQLPedidoRepository`, ela declara receber a interface `PedidoRepository`. A implementação concreta `MySQLPedidoRepository` implementa essa interface. Se amanhã o banco mudar para MongoDB ou PostgreSQL, as regras de negócio não sofrerão uma única linha de modificação.
</details>
