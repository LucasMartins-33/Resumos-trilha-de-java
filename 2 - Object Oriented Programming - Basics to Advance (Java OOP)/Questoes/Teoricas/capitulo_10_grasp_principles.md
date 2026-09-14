# Questões Teóricas: Capítulo 10 - Princípios GRASP

1. **GRASP vs SOLID:** Diferente dos princípios SOLID, os padrões GRASP têm um foco um pouco mais pragmático e direcionado a um problema específico de modelagem OOP. Qual é esse foco principal dos padrões GRASP?
<details>
<summary>👀 Ver Resposta</summary>

O foco primordial dos padrões GRASP (*General Responsibility Assignment Software Patterns*) é orientar os desenvolvedores na **atribuição de responsabilidades** a classes e objetos durante a modelagem orientada a objetos: responder à pergunta fundamental *"Qual classe deve conter este método, dado ou comportamento?"*. Enquanto o SOLID é voltado para arquitetura e manutenibilidade estrutural, o GRASP orienta a dinâmica de interação e o desenho detalhado das classes.
</details>

2. **Information Expert (Especialista na Informação):** Qual é a regra básica deste padrão para atribuir responsabilidades a uma classe?
<details>
<summary>👀 Ver Resposta</summary>

A regra básica é: **atribua a responsabilidade à classe que possui a informação necessária para cumpri-la**. Por exemplo, se precisamos calcular o valor total de uma venda, a responsabilidade deve ser da classe `Pedido` (ou `ItemPedido`), pois são elas que conhecem os itens, quantidades e preços unitários dos produtos, evitando que classes externas fiquem "roubando" dados para fazer contas que a própria classe deveria fazer.
</details>

3. **Creator (Criador):** Segundo o padrão Creator, quais as condições ideais para que a classe "A" seja responsável por instanciar a classe "B"?
<details>
<summary>👀 Ver Resposta</summary>

A classe `A` deve ser a criadora das instâncias de `B` se pelo menos uma (e idealmente várias) das seguintes condições for verdadeira:
* `A` contém ou agrega instâncias de `B`;
* `A` registra instâncias de `B`;
* `A` utiliza intensivamente instâncias de `B`;
* `A` possui os dados de inicialização necessários que serão passados para `B` no momento de sua criação.
</details>

4. **Controller (Controlador):** O que é um Controller no GRASP e qual é sua responsabilidade quando intercepta solicitações vindas da camada de interface do usuário (UI)?
<details>
<summary>👀 Ver Resposta</summary>

Um Controller é o primeiro objeto além da camada de UI que recebe e coordena um evento de operação do sistema. Sua responsabilidade não é executar a regra de negócio diretamente, mas sim atuar como um **coordenador**: ele recebe a requisição, delega o trabalho pesado para os objetos de domínio especialistas corretos e controla o fluxo de resposta da operação.
</details>

5. **Low Coupling (Baixo Acoplamento):** Como um princípio GRASP, como o Baixo Acoplamento orienta nossas decisões quando estamos divididos entre duas soluções de design que resolvem o mesmo problema?
<details>
<summary>👀 Ver Resposta</summary>

Ele atua como um critério de avaliação avaliando o impacto da decisão no sistema. Quando duas opções de design cumprem a mesma tarefa funcional, o princípio orienta a escolher aquela que crie o **menor número de dependências**, que reduza a probabilidade de alterações em cascata e que aumente o potencial de reutilização e facilidade de testes dos componentes envolvidos.
</details>

6. **High Cohesion (Alta Coesão):** Em conjunto com o baixo acoplamento, o que acontece com uma classe se falharmos em manter sua coesão alta? (Como ela se parece ou se comporta?)
<details>
<summary>👀 Ver Resposta</summary>

A classe se transforma em uma "God Class" ou "Blob": ela fica desproporcionalmente grande, difícil de ler e entender, assume responsabilidades que pertencem a outros domínios e passa a manter dependências com dezenas de outros módulos. Qualquer alteração nela se torna arriscada e propensa a efeitos colaterais.
</details>

7. **Polymorphism (Polimorfismo):** O GRASP também lista o Polimorfismo. Como ele difere do conceito básico de linguagem de programação e se torna um padrão de delegação de comportamento condicional?
<details>
<summary>👀 Ver Resposta</summary>

No contexto do GRASP, o Polimorfismo é utilizado como uma diretriz de design para **eliminar estruturas condicionais complexas (`if/else` ou `switch`) baseadas no tipo de um objeto**. Em vez de o chamador checar o tipo do objeto para decidir o que executar, cria-se uma operação polimórfica comum através de uma interface ou superclasse, delegando para cada subclasse a responsabilidade de executar seu próprio comportamento variante.
</details>

8. **Pure Fabrication (Fabricação Pura):** Às vezes, o *Information Expert* nos leva a misturar lógicas de banco de dados com entidades de negócio. Como o padrão *Pure Fabrication* (ex: Repositórios, DAOs) resolve isso?
<details>
<summary>👀 Ver Resposta</summary>

O padrão Pure Fabrication cria uma classe artificial de conveniência que não representa nenhum conceito do domínio do mundo real, mas é inventada pelo arquiteto para manter a alta coesão e o baixo acoplamento. Exemplos clássicos são classes como `UsuarioRepository`, `GeradorDeRelatorioPDF` ou `LogService`. Elas evitam que a entidade de domínio `Usuario` seja poluída com chamadas SQL ou formatações de arquivos, mantendo as entidades puras.
</details>

9. **Indirection (Indireção):** Qual é o objetivo do padrão Indirection (ex: criar um intermediário/adaptador) na redução de acoplamento entre dois componentes fortes?
<details>
<summary>👀 Ver Resposta</summary>

O objetivo é evitar o acoplamento direto entre dois ou mais componentes introduzindo um **objeto mediador ou intermediário** entre eles. Dessa forma, as partes interagem através do intermediário sem precisarem se conhecer diretamente, permitindo que qualquer um dos lados evolua ou seja substituído sem exigir alterações estruturais no outro componente.
</details>

10. **Protected Variations (Variações Protegidas):** O que é este padrão e como as Interfaces em Java ajudam a proteger o sistema de variações, atualizações e dependências instáveis?
<details>
<summary>👀 Ver Resposta</summary>

O padrão Protected Variations orienta a identificar pontos de instabilidade ou variação previsível no sistema e envolvê-los com uma **interface estável**. As interfaces em Java atuam como contratos fixos: o sistema consome a interface e fica imune às variações das implementações concretas que residem por trás dela. Se a biblioteca externa, API ou fornecedor de serviço mudar, apenas a classe adaptadora é atualizada, mantendo todo o resto do sistema protegido.
</details>
