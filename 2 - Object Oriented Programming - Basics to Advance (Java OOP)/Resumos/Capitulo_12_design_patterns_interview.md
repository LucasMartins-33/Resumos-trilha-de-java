# Anotações de Estudo: Java Core - Capítulo 12 (Design Patterns Interview)

> [!NOTE]
> Este capítulo funciona como um guia de preparação para entrevistas focadas em Design Patterns e Princípios de Orientação a Objetos. É uma compilação densa dos conceitos aprendidos nos capítulos anteriores (SOLID, GRASP, GoF).

---

## 1. Padrões de Projeto (Design Patterns - GoF)

**Q: O que são Design Patterns?**
São soluções reutilizáveis para problemas comuns encontrados no desenvolvimento de software.

**Q: De quais elementos um Design Pattern é composto?**
1. **Nome**: Ajuda a comunicar rapidamente o problema e a solução.
2. **Objetivo (Goal)**: O escopo do problema a ser resolvido.
3. **Solução**: Uma descrição abstrata das classes e suas interações.
4. **Resultados (Consequências)**: Os prós e contras de aplicar a solução.

**Q: Quais são as três categorias dos Padrões GoF?**
- **Criacionais**: Focam no processo de criação/instanciação de objetos (Abstract Factory, Factory Method, Builder, Singleton, Prototype).
- **Estruturais**: Focam na composição de classes/objetos para formar estruturas maiores e separá-las (Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy).
- **Comportamentais**: Focam em como as classes interagem e distribuem responsabilidades (Chain of Responsibility, Command, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor, Interpreter).

---

## 2. Princípios de Design Essenciais

**Q: Explique DRY, KISS e YAGNI.**
* **DRY (Don't Repeat Yourself)**: Reduzir duplicação de código utilizando abstrações. Não escreva a mesma lógica em vários lugares.
* **KISS (Keep It Simple, Stupid)**: Declara a simplicidade como objetivo primário. Não use ferramentas ou lógicas mais complexas do que o necessário.
* **YAGNI (You Ain't Gonna Need It)**: Implemente apenas o que foi definido/requisitado para agora. Rejeite adicionar funcionalidades redundantes ou pensando em um "futuro distante" que talvez nunca chegue.

**Q: O que são Condições Yoda (Yoda Conditions)?**
É um estilo de escrita de condições muito usado em C/C++ (mas também visto em Java) onde a constante é colocada à esquerda do operador: `if (5 == a)` em vez de `if (a == 5)`. Isso previne o erro comum de atribuição acidental `if (a = 5)`, gerando um erro de compilação imediato. 

**Q: O que são Mapas CRC?**
*Class-Responsibility-Collaboration (CRC) Cards* é um método de *brainstorming* para desenhar software O.O. Ajuda os designers a focarem na essência da classe (suas responsabilidades e quem ela colabora) escondendo detalhes de implementação num primeiro momento. Ajuda a prevenir o erro de dar responsabilidades demais para uma única classe.

---

## 3. Anti-Patterns (O que NÃO fazer)

**Q: Quais Anti-patterns você conhece?**
* **Big Ball of Mud (Grande Bola de Lama)**: Um sistema sem a menor arquitetura discernível. 
* **Yo-Yo Problem**: Ocorre em hierarquias de herança muito profundas onde você precisa ficar navegando infinitamente "para cima e para baixo" entre classes mães e filhas para entender o fluxo de uma simples chamada.
* **Magic Button**: Lógica de negócios pesada e não estruturada diretamente acoplada a um evento de UI (ex: `onClick` fazendo cálculos, chamadas a DB e enviando e-mail).
* **Magic Number / Magic String**: Números ou strings fixas e repetidas pelo código sem nenhuma explicação ou constante definida (ex: usar `3.14` direto na fórmula ao invés de `Math.PI`).
* **Gas Factory (Fábrica de Gás)**: Desenvolver um design absurdamente complexo para resolver uma tarefa incrivelmente simples. (Viola o princípio KISS).
* **Analysis Paralysis (Paralisia por Análise)**: Passar meses apenas planejando e desenhando o sistema, impedindo que qualquer código real seja entregue.
* **Interface Bloat**: Interfaces inchadas e gigantescas tentando cobrir todas as operações possíveis (Viola o princípio *Interface Segregation* do SOLID).

---

## 4. O.O. Paradigmas (OOAD, OOD, OOA)

* **OOA (Object-Oriented Analysis)**: O foco é entender os requisitos e o domínio do negócio através da lente de objetos/entidades (o "O QUÊ").
* **OOD (Object-Oriented Design)**: A transição da análise para a solução lógica, definindo métodos, atributos e modelos do sistema (o "COMO").
* **OOAD (Object-Oriented Analysis and Design)**: A disciplina conjunta de descobrir os objetos do problema (OOA) e desenhar como eles irão interagir para resolvê-lo (OOD).

---

## 5. Recapitulação Rápida: SOLID e GRASP

**SOLID**: Princípios fundamentais de design a nível de código.
* **SRP**: Uma única responsabilidade (ou único motivo para mudar) por classe.
* **OCP**: Aberto para extensão (adicionar novos comportamentos), fechado para modificação (não alterar código já testado).
* **LSP**: Subtipos devem ser substituíveis por seus tipos base sem quebrar o sistema.
* **ISP**: Muitas interfaces pequenas e focadas são melhores que uma interface "faz-tudo".
* **DIP**: Dependa de abstrações (interfaces), não de classes concretas.

**GRASP**: Padrões de atribuição de responsabilidades a nível arquitetural.
* **Information Expert**: Aja onde os dados estão.
* **Creator**: Quem agrega os dados, cria o objeto.
* **Controller**: Lida com eventos de entrada do sistema.
* **Low Coupling & High Cohesion**: Mantenha acoplamento baixo e classes bem focadas.
* **Polymorphism**: Sem `ifs/switches` para verificar tipo; use abstrações dinâmicas.
* **Pure Fabrication**: Classes artificiais (Services/Repositories) para focar lógica sem poluir o Domínio.
* **Indirection**: Intermediários (Mediators) para reduzir acoplamento direto.
* **Protected Variations**: Proteger o sistema de variações externas criando pontos fixos (interfaces) de comunicação.
