# Questões Teóricas - Java Core (OOP, Arquitetura e Padrões)

Aqui estão as 10 questões de revisão para testar seus conhecimentos teóricos consolidados durante o curso:

### Fundamentos OOP e Java
**1.** Qual é a diferença fundamental entre Sobrecarga (*Overloading*) e Sobrescrita (*Overriding*) de métodos em Java, e como os conceitos de polimorfismo (tempo de compilação vs. tempo de execução) se aplicam a eles?

**2.** Em Java, costumamos dizer que a passagem de parâmetros é **estritamente "por valor"** (*pass-by-value*). No entanto, quando passamos um objeto para um método, conseguimos alterar seus atributos internos. Explique teoricamente por que isso acontece sem violar a regra do "pass-by-value".

### Arquitetura e Clean Code
**3.** Em termos de design de software, explique o que significa buscar **"Alta Coesão e Baixo Acoplamento"**. Por que essa combinação é considerada a meta de ouro de uma boa arquitetura?

**4.** O Princípio da Inversão de Dependência (**D** do SOLID) afirma que "módulos de alto nível não devem depender de módulos de baixo nível, ambos devem depender de abstrações". Na prática do dia a dia, como aplicamos esse princípio no código?

### Design Patterns (GRASP & GoF)
**5.** Segundo o padrão GRASP **Information Expert** (Especialista na Informação), qual é o critério exato que devemos usar para decidir a qual classe devemos atribuir uma determinada responsabilidade ou método?

**6.** Qual é a principal diferença de intenção entre os padrões GoF **Strategy** e **State**? (Considere que ambos servem para alterar o comportamento em tempo de execução e possuem diagramas UML praticamente idênticos).

### Modelagem UML
**7.** Em um Diagrama de Classes UML, qual é a diferença semântica e do ciclo de vida entre os relacionamentos de **Agregação** (losango vazio) e **Composição** (losango preenchido)?

**8.** Em um Diagrama de Casos de Uso, qual é a diferença prática entre os relacionamentos `<<include>>` e `<<extend>>` quando um caso de uso aponta para o outro?

### SOA, Eventos e Sistemas Distribuídos
**9.** Na Arquitetura Orientada a Eventos (EDA), por que é fundamental fazer a distinção entre um **Domain Event** (Evento de Domínio) e um **Integration Event** (Evento de Integração)? O que acontece se enviarmos eventos de domínio diretamente para o mundo externo?

**10.** Transações distribuídas tradicionais (como *Two-Phase Commit*) não escalam bem em arquiteturas de microserviços. Explique como o padrão **Saga** lida com processos de negócio que falham pela metade e explique o papel das chamadas **Ações de Compensação** (*Compensations*).
