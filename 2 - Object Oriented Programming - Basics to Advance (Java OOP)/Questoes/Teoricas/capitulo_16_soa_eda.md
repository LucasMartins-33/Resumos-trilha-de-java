# Questões Teóricas: Capítulo 16 - SOA, EDA e Sistemas Distribuídos

1. **Conceito de SOA Moderno:** SOA hoje não se refere a velhos servidores SOAP de mercado, mas a princípios arquiteturais. O que significa definir um serviço não como um "artefato de deploy", mas sim como uma "Fronteira de Responsabilidade" (*Boundary*)?
<details>
<summary>👀 Ver Resposta</summary>

Significa que a essência de um serviço não reside na forma como ele é empacotado ou implantado (um JAR, contêiner ou função serverless), mas sim na **fronteira de autoridade sobre um domínio de negócio específico**. Um serviço é o dono exclusivo de seus dados, regras e lógicas correspondentes àquele domínio; nenhum outro componente pode violar essa fronteira sem passar por suas interfaces públicas.
</details>

2. **Autonomia (Ownership):** O que deve acontecer quando quebramos a regra primária de autonomia, permitindo que dois serviços acessem ou compartilhem diretamente o mesmo Banco de Dados?
<details>
<summary>👀 Ver Resposta</summary>

Gera-se o perigoso anti-pattern **Shared Database**, destruindo a autonomia e o isolamento dos serviços. O acoplamento passa a ocorrer no nível do banco de dados: qualquer alteração de schema, tabela ou índice feita por um serviço pode derrubar o outro sem aviso prévio. Além disso, elimina a possibilidade de escalabilidade independente e de escolha de tecnologias de persistência apropriadas para cada contexto de domínio.
</details>

3. **Contratos (Contracts):** Por que em microsserviços o Contrato é a API real? O que acontece com os serviços clientes se começarmos a vazar (expor) nossas Entidades Internas de Domínio no JSON da nossa API pública?
<details>
<summary>👀 Ver Resposta</summary>

O contrato (seja OpenAPI/REST, Protobuf ou Avro) é a única interface pública formal de comunicação. Se expusermos diretamente as Entidades Internas de Domínio no payload da API, criamos um **acoplamento temporal e de implementação severo**: qualquer refatoração interna, alteração de atributos ou ajuste de modelo de negócio quebrará os clientes externos imediatamente. Deve-se sempre expor DTOs específicos de contrato desacoplados do modelo interno.
</details>

4. **Request/Response (Sync) em Escala:** Em sistemas distribuídos, quais os grandes perigos de se construir cadeias síncronas longas (Serviço A chama B, que chama C, que chama D), também conhecidas como falhas em cascata?
<details>
<summary>👀 Ver Resposta</summary>

Os grandes perigos são:
1. **Falhas em Cascata:** Se o serviço D ficar lento ou falhar, a fila de chamadas trava os serviços C, B e A, esgotando pools de conexões e derrubando toda a cadeia (*Cascading Failure*);
2. **Latência Acumulada:** O tempo de resposta final do serviço A é a soma das latências de rede e processamento de todos os serviços subsequentes;
3. **Disponibilidade Comprometida:** A disponibilidade do sistema passa a ser a multiplicação das disponibilidades de cada nó envolvido (se cada serviço tiver 99% de uptime, uma cadeia de 4 serviços terá apenas cerca de 96% de uptime).
</details>

5. **Eventos (Events):** Qual é a diferença arquitetural e filosófica entre um "Comando" (*canceleOPedido()*) e um "Evento" (*PedidoCancelado*) na perspectiva de quem os envia?
<details>
<summary>👀 Ver Resposta</summary>

* **Comando:** É uma **intenção imperativa dirigida**. O emissor envia uma ordem para um destinatário específico com a expectativa clara de que uma ação seja realizada (e geralmente espera saber o resultado). Há um acoplamento semântico entre o emissor e o receptor.
* **Evento:** É uma **notificação de um fato passado e imutável**. O emissor simplesmente anuncia para o mundo *"algo aconteceu no meu domínio"* (ex: `PedidoCriado`), sem saber nem se importar com quem consumirá essa informação ou o que fará com ela, proporcionando desacoplamento total.
</details>

6. **Domain Events vs Integration Events:** Por que não devemos jogar Eventos de Domínio brutos (as validações que acontecem internamente no negócio do nosso serviço) diretamente em um Message Broker para outros sistemas lerem?
<details>
<summary>👀 Ver Resposta</summary>

Porque **Eventos de Domínio** são entidades internas, granulares e focadas nas regras fechadas de um único microsserviço (Bounded Context). Publicá-los diretamente em um Message Broker expõe a modelagem interna do serviço para terceiros, gerando alto acoplamento e quebra de contratos a cada refatoração. Deve-se publicar **Integration Events** (Eventos de Integração), que são contratos públicos estáveis e sanitizados, desenhados especificamente para consumo por outros subsistemas.
</details>

7. **Garantias de Entrega (Delivery Guarantees):** Explique o que é a entrega *At least once* (Pelo menos uma vez) provida pela maioria dos brokers (Kafka, Rabbit) e porque ela força o consumidor a implementar a propriedade de Idempotência.
<details>
<summary>👀 Ver Resposta</summary>

A garantia *At least once* assegura que nenhuma mensagem será perdida em trânsito; contudo, em casos de oscilação de rede, rebalanceamento de partições ou falhas no recebimento de confirmações (*acks*), a mesma mensagem pode ser retransmitida e entregue mais de uma vez. Por essa razão, os consumidores devem ser **idempotentes**, ou seja, processar a mesma mensagem repetidas vezes deve produzir exatamente o mesmo efeito que processá-la uma única vez (usando, por exemplo, controle de IDs de mensagens processadas no banco).
</details>

8. **Coreografia vs Orquestração:** Em processos de negócio distribuídos, a Coreografia promete alta autonomia e descentralização, mas introduz um risco invisível. Qual é o principal perigo de usar apenas Coreografia conforme o sistema cresce (o famoso "Distributed Big Ball of Mud")?
<details>
<summary>👀 Ver Resposta</summary>

O perigo é a **perda de visibilidade e controle do fluxo de negócio**. Em uma coreografia pura em larga escala, os serviços reagem cegamente a eventos uns dos outros em cadeia; não existe um local central onde o fluxo do processo esteja documentado ou monitorável. Quando ocorre um erro ou comportamento inesperado, é extremamente difícil rastrear em qual elo da cadeia de eventos a transação se perdeu, gerando o caos arquitetural conhecido como "Lamaçal Distribuído".
</details>

9. **Padrão Saga:** Como o padrão Saga lida com falhas em processos distribuídos transacionais sem tentar realizar *Rollbacks* simultâneos nos bancos de dados alheios?
<details>
<summary>👀 Ver Resposta</summary>

Como transações ACID clássicas (Two-Phase Commit) não escalam em ambientes distribuídos, o padrão Saga decompõe a transação global em uma série de **transações locais independentes** em cada microsserviço. Se uma das etapas falhar no meio do processo (ex: pagamento recusado após o estoque ter sido reservado), a Saga executa uma sequência ordenada de **Transações de Compensação** (*Compensating Transactions*) que revertem semanticamente os efeitos das etapas que já haviam sido concluídas (ex: cancelando a reserva do estoque).
</details>

10. **O Problema do Dual Write e o Outbox Pattern:** O que acontece quando tentamos salvar algo no banco e logo em seguida emitir um evento num broker dentro do código padrão de um microsserviço? Explique como a tabela *Outbox* resolve as possíveis inconsistências geradas se a rede oscilar entre esses dois passos.
<details>
<summary>👀 Ver Resposta</summary>

O problema do **Dual Write** ocorre porque salvar no banco e publicar no broker são duas operações em sistemas externos distintos que não compartilham a mesma transação. Se o banco salvar com sucesso, mas a rede cair antes de publicar no broker, o evento se perde e o sistema fica inconsistente.
O **Transactional Outbox Pattern** resolve isso salvando a entidade de negócio E o evento pendente em uma tabela `Outbox` **dentro da mesmíssima transação ACID do banco de dados local**. Um processo em segundo plano (como Debezium via CDC ou um job agendado) lê a tabela Outbox e despacha as mensagens para o broker de forma confiável e assíncrona.
</details>
