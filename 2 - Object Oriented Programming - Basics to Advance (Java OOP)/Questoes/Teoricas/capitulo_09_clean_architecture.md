# Questões Teóricas: Capítulo 9 - Arquitetura Limpa (Clean Architecture)

1. **Objetivo da Clean Architecture:** Qual é o principal problema que a Arquitetura Limpa tenta resolver em relação aos sistemas monolíticos acoplados tradicionais?
2. **Tipos de Acoplamento:** Qual é a diferença entre *Acoplamento Aferente* (quem depende de mim) e *Acoplamento Eferente* (de quem eu dependo)? Qual deles devemos buscar minimizar nas regras de negócio principais?
3. **Coesão Funcional vs Lógica:** Por que a Coesão Funcional (tudo que trabalha junto para a mesma tarefa) é superior à Coesão Lógica (agrupar métodos apenas porque são da mesma "categoria", ex: classe `Math`)?
4. **O Paradigma do Plugin:** O que significa dizer que os detalhes (Banco de Dados, UI, Web) devem ser "plugins" para as Regras de Negócio?
5. **A Regra da Dependência (Dependency Rule):** Na representação de camadas em círculos concêntricos da Clean Architecture, para qual direção todas as setas de dependência no código fonte devem apontar?
6. **Entidades (Entities):** Qual tipo de lógica deve ser alocada na camada mais interna (Entidades) do sistema?
7. **Casos de Uso (Use Cases):** Qual é a diferença prática de responsabilidade entre a camada de Casos de Uso e a camada de Entidades?
8. **Adaptadores de Interface:** Qual é o papel principal da camada de Interface Adapters (Controllers, Presenters, Gateways)?
9. **O Banco de Dados como Detalhe:** Por que o framework de persistência ou o Banco de Dados é considerado um "detalhe externo" na Clean Architecture e não o núcleo do sistema?
10. **Desvantagens e Overengineering:** Em quais cenários a aplicação rigorosa da Clean Architecture pode ser considerada prematura ou um caso de *Overengineering*?
