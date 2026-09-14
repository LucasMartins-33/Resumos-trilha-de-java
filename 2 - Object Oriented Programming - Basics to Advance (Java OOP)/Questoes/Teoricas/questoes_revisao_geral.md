# Questões Teóricas - Java Core (OOP, Arquitetura e Padrões)

Aqui estão as 10 questões de revisão para testar seus conhecimentos teóricos consolidados durante o curso:

### Fundamentos OOP e Java
**1.** Qual é a diferença fundamental entre Sobrecarga (*Overloading*) e Sobrescrita (*Overriding*) de métodos em Java, e como os conceitos de polimorfismo (tempo de compilação vs. tempo de execução) se aplicam a eles?
<details>
<summary>👀 Ver Resposta</summary>

* **Sobrecarga (*Overloading*):** Ocorre dentro da mesma classe, onde métodos têm o mesmo nome, mas listas de parâmetros diferentes (quantidade, tipo ou ordem). Representa o **Polimorfismo em Tempo de Compilação (Estático)**, pois o compilador resolve qual método invocar analisando os tipos dos argumentos antes do programa rodar.
* **Sobrescrita (*Overriding*):** Ocorre em relações de herança, onde uma subclasse fornece uma implementação específica para um método já declarado na superclasse ou interface, mantendo rigorosamente o mesmo nome e assinatura. Representa o **Polimorfismo em Tempo de Execução (Dinâmico)**, onde a JVM decide qual versão do método executar em tempo de execução com base no tipo concreto do objeto na Heap (*Dynamic Dispatch*).
</details>

**2.** Em Java, costumamos dizer que a passagem de parâmetros é **estritamente "por valor"** (*pass-by-value*). No entanto, quando passamos um objeto para um método, conseguimos alterar seus atributos internos. Explique teoricamente por que isso acontece sem violar a regra do "pass-by-value".
<details>
<summary>👀 Ver Resposta</summary>

Isso acontece porque a variável em Java **nunca armazena o objeto em si, mas sim o seu endereço de referência na memória Heap**. Quando um objeto é passado como argumento, a JVM copia esse endereço de referência por valor e o entrega ao parâmetro do método. Como o parâmetro local e a variável original possuem cópias do mesmo endereço de referência, ambos apontam para o mesmo objeto físico na Heap. As alterações nos atributos afetam o objeto compartilhado, mas a passagem continua sendo estritamente por valor (cópia de ponteiros).
</details>

### Arquitetura e Clean Code
**3.** Em termos de design de software, explique o que significa buscar **"Alta Coesão e Baixo Acoplamento"**. Por que essa combinação é considerada a meta de ouro de uma boa arquitetura?
<details>
<summary>👀 Ver Resposta</summary>

* **Alta Coesão:** Significa que uma classe ou módulo tem responsabilidades claras e focadas em um único propósito de negócio; todos os seus elementos trabalham juntos para cumprir essa meta.
* **Baixo Acoplamento:** Significa que as classes dependem o mínimo possível dos detalhes de implementação de outros módulos, interagindo preferencialmente através de interfaces e abstrações estáveis.
Essa combinação é a meta de ouro porque produz sistemas fáceis de entender, simples de testar unitariamente com mocks, altamente reutilizáveis e resistentes a falhas em cascata quando o código precisa evoluir.
</details>

**4.** O Princípio da Inversão de Dependência (**D** do SOLID) afirma que "módulos de alto nível não devem depender de módulos de baixo nível, ambos devem depender de abstrações". Na prática do dia a dia, como aplicamos esse princípio no código?
<details>
<summary>👀 Ver Resposta</summary>

Aplicamos definindo **Interfaces Java** na camada de negócio (alto nível) que descrevem as necessidades de persistência ou comunicação externa (ex: `PedidoRepository`, `EmailSender`). As classes de serviço de negócio dependem exclusivamente dessas interfaces injetadas via construtor (Injeção de Dependências). As implementações técnicas concretas (módulos de baixo nível, como `PostgresPedidoRepository` ou `SendGridEmailSender`) implementam essas interfaces, invertendo a direção tradicional do acoplamento.
</details>

### Design Patterns (GRASP & GoF)
**5.** Segundo o padrão GRASP **Information Expert** (Especialista na Informação), qual é o critério exato que devemos usar para decidir a qual classe devemos atribuir uma determinada responsabilidade ou método?
<details>
<summary>👀 Ver Resposta</summary>

O critério exato é: **atribuir a responsabilidade à classe que possui a informação necessária para cumpri-la**. Em vez de extrair dados de um objeto para processá-los em outra classe externa (gerando acoplamento e violando o encapsulamento), colocamos a lógica de cálculo ou validação dentro da própria classe que armazena os atributos necessários para aquela operação.
</details>

**6.** Qual é a principal diferença de intenção entre os padrões GoF **Strategy** e **State**? (Considere que ambos servem para alterar o comportamento em tempo de execução e possuem diagramas UML praticamente idênticos).
<details>
<summary>👀 Ver Resposta</summary>

A diferença reside na **intenção e no controle das transições**:
* **Strategy:** O foco é encapsular algoritmos intercambiáveis e independentes. O cliente geralmente escolhe explicitamente qual estratégia injetar (ex: escolher entre cálculo de frete aéreo ou terrestre) e as estratégias não conhecem umas às outras.
* **State:** O foco é permitir que um objeto altere seu comportamento quando seu estado interno muda, como uma máquina de estados finitos. As classes de estado concreto frequentemente conhecem outros estados e coordenam ativamente as transições entre eles conforme os eventos ocorrem no contexto.
</details>

### Modelagem UML
**7.** Em um Diagrama de Classes UML, qual é a diferença semântica e do ciclo de vida entre os relacionamentos de **Agregação** (losango vazio) e **Composição** (losango preenchido)?
<details>
<summary>👀 Ver Resposta</summary>

* **Agregação (Losango Vazio):** Relação "todo/parte" fraca. Os objetos parte possuem ciclo de vida independente do todo (se o todo for destruído, as partes continuam existindo). Exemplo: `Universidade` e `Professor`.
* **Composição (Losango Preenchido):** Relação "todo/parte" forte com dependência existencial. As partes pertencem exclusivamente ao todo e não têm sentido de existir sem ele; se o todo for destruído, as partes são destruídas juntas. Exemplo: `Pedido` e `ItemDoPedido`.
</details>

**8.** Em um Diagrama de Casos de Uso, qual é a diferença prática entre os relacionamentos `<<include>>` e `<<extend>>` quando um caso de uso aponta para o outro?
<details>
<summary>👀 Ver Resposta</summary>

* **`<<include>>`:** A execução do caso de uso incluído é **obrigatória** e incondicional para que o caso de uso base seja concluído com sucesso (ex: "Realizar Compra" inclui obrigatoriamente "Autenticar Usuário").
* **`<<extend>>`:** A execução do caso de uso extensor é **opcional e condicional**, disparada apenas se uma condição específica for satisfeita em um ponto de extensão do caso de uso base (ex: "Calcular Frete Expresso" estende condicionalmente "Finalizar Compra").
</details>

### SOA, Eventos e Sistemas Distribuídos
**9.** Na Arquitetura Orientada a Eventos (EDA), por que é fundamental fazer a distinção entre um **Domain Event** (Evento de Domínio) e um **Integration Event** (Evento de Integração)? O que acontece se enviarmos eventos de domínio diretamente para o mundo externo?
<details>
<summary>👀 Ver Resposta</summary>

É fundamental porque **Eventos de Domínio** pertencem à linguagem e regras internas exclusivas de um único microsserviço (Bounded Context). Publicá-los diretamente para outros sistemas vaza detalhes internos de implementação e cria um forte acoplamento com o modelo de dados do serviço. Os **Integration Events** são contratos formais públicos, estáveis e sanitizados, desenhados para que outros microsserviços consumam informações sem que mudanças internas no serviço emissor gerem impactos colaterais.
</details>

**10.** Transações distribuídas tradicionais (como *Two-Phase Commit*) não escalam bem em arquiteturas de microserviços. Explique como o padrão **Saga** lida com processos de negócio que falham pela metade e explique o papel das chamadas **Ações de Compensação** (*Compensations*).
<details>
<summary>👀 Ver Resposta</summary>

O padrão Saga substitui a transação global distribuída por uma sequência de **transações locais independentes** executadas sequencialmente em cada microsserviço envolvido. Caso alguma etapa falhe no caminho (ex: pagamento reprovado), a Saga não pode fazer um "rollback" em bancos alheios; em vez disso, ela dispara **Ações de Compensação** (transações compensatórias) executadas em ordem inversa para reverter semanticamente os efeitos das etapas que já haviam sido confirmadas (ex: liberando produtos reservados e cancelando faturas provisórias).
</details>
