# Questões Teóricas: Capítulo 9 - Arquitetura Limpa (Clean Architecture)

1. **Objetivo da Clean Architecture:** Qual é o principal problema que a Arquitetura Limpa tenta resolver em relação aos sistemas monolíticos acoplados tradicionais?
<details>
<summary>👀 Ver Resposta</summary>

A Clean Architecture resolve o problema do forte acoplamento entre as regras de negócio e os componentes externos (frameworks, bancos de dados, interfaces com o usuário e protocolos de rede). Nos sistemas tradicionais, as regras de negócio acabam contaminadas por anotações e tipos do banco de dados ou da web, tornando o sistema refém da tecnologia, frágil a mudanças e extremamente difícil de testar de forma isolada.
</details>

2. **Tipos de Acoplamento:** Qual é a diferença entre *Acoplamento Aferente* (quem depende de mim) e *Acoplamento Eferente* (de quem eu dependo)? Qual deles devemos buscar minimizar nas regras de negócio principais?
<details>
<summary>👀 Ver Resposta</summary>

* **Acoplamento Aferente ($C_a$):** Mede o número de classes externas que dependem daquele componente (indica a responsabilidade do módulo).
* **Acoplamento Eferente ($C_e$):** Mede o número de classes externas de que aquele componente depende (indica a instabilidade do módulo).
Nas regras de negócio centrais, devemos buscar **minimizar o Acoplamento Eferente ($C_e$)**, fazendo com que o núcleo do sistema não dependa de nada externo, garantindo máxima estabilidade e independência.
</details>

3. **Coesão Funcional vs Lógica:** Por que a Coesão Funcional (tudo que trabalha junto para a mesma tarefa) é superior à Coesão Lógica (agrupar métodos apenas porque são da mesma "categoria", ex: classe `Math`)?
<details>
<summary>👀 Ver Resposta</summary>

A **Coesão Funcional** agrupa elementos que cooperam estreitamente para executar uma única atividade bem definida de negócio (ex: `ProcessadorDeMatricula`), garantindo que quando uma regra muda, apenas aquele componente é tocado. Já a **Coesão Lógica** agrupa métodos heterogêneos apenas por conveniência sintática ou temática (ex: classes utilitárias `Utils` ou `Math`), atraindo dependências desconexas e múltiplos motivos para mudar, o que viola o SRP e gera alto risco de regressão.
</details>

4. **O Paradigma do Plugin:** O que significa dizer que os detalhes (Banco de Dados, UI, Web) devem ser "plugins" para as Regras de Negócio?
<details>
<summary>👀 Ver Resposta</summary>

Significa que o núcleo de regras de negócio é o componente central e autossuficiente do software. Tecnologias externas como o banco de dados (PostgreSQL, MongoDB), interfaces gráficas (React, JavaFX, CLI) ou protocolos de comunicação (REST, gRPC) são tratados como extensões substituíveis ("plugins") que se conectam ao núcleo através de interfaces/portas. A aplicação não sabe nem se importa com qual banco ou framework está conectado em um determinado momento.
</details>

5. **A Regra da Dependência (Dependency Rule):** Na representação de camadas em círculos concêntricos da Clean Architecture, para qual direção todas as setas de dependência no código fonte devem apontar?
<details>
<summary>👀 Ver Resposta</summary>

Todas as setas de dependência de código fonte devem apontar **estritamente para dentro**, em direção às políticas de mais alto nível (o domínio central). Nenhuma linha de código em um círculo interno pode ter conhecimento de nada pertencente a um círculo mais externo (nenhum tipo, classe, framework ou anotação da camada externa pode ser importado pelas camadas internas).
</details>

6. **Entidades (Entities):** Qual tipo de lógica deve ser alocada na camada mais interna (Entidades) do sistema?
<details>
<summary>👀 Ver Resposta</summary>

As Entidades encapsulam as **Regras de Negócio Empresariais Críticas** (*Enterprise-Wide Business Rules*). São os conceitos, dados e validações fundamentais do negócio que existiriam mesmo se a empresa não tivesse computadores ou softwares (como regras de cálculo de juros, validação de dados de um cliente ou regras de elegibilidade de contratos). Elas são 100% agnósticas a frameworks, bancos e telas.
</details>

7. **Casos de Uso (Use Cases):** Qual é a diferença prática de responsabilidade entre a camada de Casos de Uso e a camada de Entidades?
<details>
<summary>👀 Ver Resposta</summary>

Enquanto as Entidades guardam regras universais e atemporais da empresa, os **Casos de Uso** orquestram as **Regras de Negócio Específicas da Aplicação**. Um Caso de Uso orquestra o fluxo de dados entre as Entidades e os adaptadores externos, recebendo dados de entrada, aplicando a sequência de passos necessária para concluir uma operação do sistema (como "Finalizar Pedido" ou "Emitir Boleto") e devolvendo a resposta para os apresentadores.
</details>

8. **Adaptadores de Interface:** Qual é o papel principal da camada de Interface Adapters (Controllers, Presenters, Gateways)?
<details>
<summary>👀 Ver Resposta</summary>

O papel dessa camada é atuar como uma **ponte de tradução bidirecional** de dados. Ela converte os dados no formato mais conveniente para entidades e casos de uso para o formato mais conveniente para agentes externos (como converter um JSON de uma requisição HTTP em um objeto DTO de entrada do Caso de Uso, ou converter a resposta de negócio em um ViewModel ou status HTTP formatado para o usuário).
</details>

9. **O Banco de Dados como Detalhe:** Por que o framework de persistência ou o Banco de Dados é considerado um "detalhe externo" na Clean Architecture e não o núcleo do sistema?
<details>
<summary>👀 Ver Resposta</summary>

Porque o banco de dados é apenas uma ferramenta mecânica de armazenamento e recuperação de dados na memória persistente (disco/rede). A essência, o valor e a lógica do sistema residem nas regras de negócio. Modelar uma aplicação partindo de tabelas do banco de dados gera um acoplamento precoce e distorce o modelo de domínio; ao isolar a persistência como um detalhe através do padrão Repository, a aplicação ganha flexibilidade para mudar schemas, drivers ou até o tipo de banco sem impactar a lógica de negócio.
</details>

10. **Desvantagens e Overengineering:** Em quais cenários a aplicação rigorosa da Clean Architecture pode ser considerada prematura ou um caso de *Overengineering*?
<details>
<summary>👀 Ver Resposta</summary>

Em sistemas pequenos, microsserviços puramente CRUD (que apenas salvam e leem dados sem regras de negócio complexas), protótipos rápidos (*PoCs*) ou MVPs com prazos curtíssimos. Nesses cenários, a Clean Architecture introduz uma quantidade substancial de cerimônia, muitas camadas, mapeamentos repetitivos entre objetos de domínio e DTOs, e dezenas de interfaces desnecessárias, gerando custos de desenvolvimento e complexidade que não trazem benefícios proporcionais.
</details>
