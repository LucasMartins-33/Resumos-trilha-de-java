# Questões Teóricas: Capítulo 12 - Entrevista sobre Design Patterns

1. **Composição sobre Herança:** Os padrões GoF são famosos pela máxima "Favoreça a composição de objetos sobre a herança de classes". Quais são as maiores desvantagens da herança de classes que justificam essa afirmação?
2. **Anti-patterns:** O que são *Anti-patterns* e qual é o impacto do chamado "God Object" ou "Blob" em um sistema?
3. **Singleton e Concorrência:** O Singleton clássico muitas vezes falha em ambientes multi-thread. Qual é a principal armadilha técnica (race condition) ao implementar um Singleton de forma ingênua?
4. **Hollywood Principle (Inversão de Controle):** O padrão **Template Method** implementa a máxima "Não nos chame, nós chamaremos você". Como ele consegue forçar a ordem do algoritmo e ainda assim permitir customização pelas subclasses?
5. **Bridge vs Adapter:** A intenção do *Adapter* é fazer duas coisas prontas funcionarem juntas após o design. Já o padrão *Bridge* é projetado *antes*. O que o padrão Bridge tenta separar?
6. **Padrão Command:** Como o padrão Command transforma uma "chamada de método" em um objeto tangível, e como isso facilita a implementação de históricos e funcionalidades de "Undo/Redo"?
7. **Flyweight vs Object Pool:** Ambos economizam memória/recursos. A diferença é que um *Pool* lida com instâncias que não podem ser compartilhadas simultaneamente, enquanto o *Flyweight* lida com compartilhamento estrutural. Explique como o Flyweight separa estados intrínsecos e extrínsecos.
8. **Padrões e OCP:** Como o padrão **Strategy** adere perfeitamente ao Princípio do Aberto/Fechado (OCP) do SOLID na prática?
9. **Mediator:** Como o padrão Mediator alivia a comunicação caótica ("teia de aranha" - *many-to-many*) entre objetos, tornando os relacionamentos uma estrela (*star topology*)?
10. **Sindrome do Arquiteto:** Em que situações a aplicação de um Design Pattern complexo (como uma arquitetura de múltiplos Factories e Decorators) é na verdade uma má escolha?
