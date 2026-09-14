# Questões Teóricas: Capítulo 16 - SOA, EDA e Sistemas Distribuídos

1. **Conceito de SOA Moderno:** SOA hoje não se refere a velhos servidores SOAP de mercado, mas a princípios arquiteturais. O que significa definir um serviço não como um "artefato de deploy", mas sim como uma "Fronteira de Responsabilidade" (*Boundary*)?
2. **Autonomia (Ownership):** O que deve acontecer quando quebramos a regra primária de autonomia, permitindo que dois serviços acessem ou compartilhem diretamente o mesmo Banco de Dados?
3. **Contratos (Contracts):** Por que em microsserviços o Contrato é a API real? O que acontece com os serviços clientes se começarmos a vazar (expor) nossas Entidades Internas de Domínio no JSON da nossa API pública?
4. **Request/Response (Sync) em Escala:** Em sistemas distribuídos, quais os grandes perigos de se construir cadeias síncronas longas (Serviço A chama B, que chama C, que chama D), também conhecidas como falhas em cascata?
5. **Eventos (Events):** Qual é a diferença arquitetural e filosófica entre um "Comando" (*canceleOPedido()*) e um "Evento" (*PedidoCancelado*) na perspectiva de quem os envia?
6. **Domain Events vs Integration Events:** Por que não devemos jogar Eventos de Domínio brutos (as validações que acontecem internamente no negócio do nosso serviço) diretamente em um Message Broker para outros sistemas lerem?
7. **Garantias de Entrega (Delivery Guarantees):** Explique o que é a entrega *At least once* (Pelo menos uma vez) provida pela maioria dos brokers (Kafka, Rabbit) e porque ela força o consumidor a implementar a propriedade de Idempotência.
8. **Coreografia vs Orquestração:** Em processos de negócio distribuídos, a Coreografia promete alta autonomia e descentralização, mas introduz um risco invisível. Qual é o principal perigo de usar apenas Coreografia conforme o sistema cresce (o famoso "Distributed Big Ball of Mud")?
9. **Padrão Saga:** Como o padrão Saga lida com falhas em processos distribuídos transacionais sem tentar realizar *Rollbacks* simultâneos nos bancos de dados alheios?
10. **O Problema do Dual Write e o Outbox Pattern:** O que acontece quando tentamos salvar algo no banco e logo em seguida emitir um evento num broker dentro do código padrão de um microsserviço? Explique como a tabela *Outbox* resolve as possíveis inconsistências geradas se a rede oscilar entre esses dois passos.
