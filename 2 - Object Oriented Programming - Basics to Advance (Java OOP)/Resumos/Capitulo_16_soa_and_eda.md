# Anotações de Estudo: Java Core - Capítulo 16 (SOA & EDA)

> [!NOTE]
> Este capítulo foca em **Arquitetura Orientada a Serviços (SOA)** e **Arquitetura Orientada a Eventos (EDA)** modernas. A ideia não é focar em ferramentas específicas (como RabbitMQ ou Kafka), mas sim nos **princípios arquiteturais** que garantem que sistemas distribuídos não entrem em colapso conforme crescem em escala.

---

## 1. SOA e EDA: Fundamentos Modernos
* **Service-Oriented Architecture (SOA)**: Originalmente associado a tecnologias como SOAP/XML, hoje SOA foca na organização do sistema em torno de **capacidades de negócio**. Um serviço é uma **fronteira de responsabilidade**, não necessariamente uma unidade de deploy.
* **Event-Driven Architecture (EDA)**: É o modelo de comunicação onde os serviços publicam e reagem a **eventos**. Reduz o acoplamento temporal.
* **Microservices**: É apenas uma escolha de *deploy* e operação. Microserviços sem os princípios de SOA viram um "Monolito Distribuído".

### Autonomia e Fronteiras (Boundaries)
Um serviço DEVE possuir:
1. **Dados (Data)**: Apenas o serviço altera seus dados.
2. **Comportamento (Behavior)**: Regras de negócio ficam contidas na fronteira.
3. **Ciclo de Vida (Lifecycle)**: Pode ser refatorado ou alterado sem quebrar o resto do sistema.

---

## 2. Contratos e Evolução
O verdadeiro "API" de um serviço não é o endpoint REST, mas o seu **Contrato**.
* **Alta Coesão, Baixo Acoplamento**: Tudo relacionado fica junto dentro do serviço; serviços interagem de forma opaca com os outros.
* **Domain Events vs Integration Events**:
  * *Domain Events*: Ficam dentro do serviço. São o resultado interno de uma regra de negócio validada.
  * *Integration Events*: São os eventos publicados para o mundo externo. Os *Domain Events* devem ser traduzidos para *Integration Events* para evitar vazar complexidade interna.
* **Evolução de Schema**: Eventos são como APIs públicas. A regra de ouro da evolução é a **Mudança Aditiva** (adicionar campos opcionais). Evite remover campos ou mudar a semântica deles, pois quebra retrocompatibilidade.

---

## 3. Comunicação: Sync vs Async
* **Sync (Request/Response)**: Acopla serviços no tempo. A latência acumula. Falhas geram *cascading failures* (falhas em cascata). Ideal apenas quando é estritamente necessário uma validação imediata (ex: no limite do sistema, lidando direto com o usuário).
* **Async (Eventos)**: Desacopla no tempo. Isolamento de falha é muito maior. Introduz o conceito de **Consistência Eventual**.

---

## 4. Garantias de Entrega e Idempotência
A infraestrutura real de rede falha constantemente. Modelos de entrega:
* **At most once (No máximo uma vez)**: Pode haver perda de dados.
* **At least once (Pelo menos uma vez)**: O mais comum. Garante entrega, mas **podem ocorrer eventos duplicados** (devido a retentativas da rede).
* **Exactly once**: Muito custoso e complexo, beira à ilusão em larga escala.

### Idempotência
Como a rede usa *At least once*, o **consumidor** deve ser idempotente. Processar o mesmo evento 2, 3 ou 10 vezes deve gerar **o mesmo estado final** que processar apenas 1 vez (geralmente checando IDs únicos no banco antes de prosseguir).

### Dead Letter Queues (DLQ) & Poison Messages
* **Poison Message**: Uma mensagem malformada ou que quebra regras de negócio. Retentá-la não vai consertá-la.
* **DLQ**: É a fila para onde essas mensagens venenosas vão após X retentativas, para não bloquearem o processamento normal do sistema.

---

## 5. Coreografia vs Orquestração
Como coordenar um processo de negócio que cruza vários serviços?
1. **Event Choreography (Coreografia)**: Descentralizado. Serviços reagem a eventos uns dos outros.
   * *Pró*: Altamente autônomo e extensível.
   * *Contra*: Difícil visibilidade. Risco de virar um "Big Ball of Mud" (dependências implícitas difíceis de debugar).
2. **Orchestration (Orquestração)**: Centralizado. Um componente dita o fluxo, dizendo "Faça A, depois B, depois C".
   * *Pró*: O fluxo é explícito e fácil de monitorar/alterar.
   * *Contra*: O orquestrador pode virar um gargalo ou um componente "Deus".

> [!TIP]
> Sistemas robustos misturam os dois. Usam **Orquestração** no caminho crítico (para ter controle) e **Coreografia** para *side-effects* (ex: enviar email, atualizar analytics).

---

## 6. Sagas e Consistência Eventual
Transações Distribuídas tradicionais (como Two-Phase Commit - 2PC) **não escalam**. Elas travam o sistema inteiro esperando confirmação.

A solução é usar o padrão **Saga**:
* Quebra uma transação grande em múltiplos **passos locais**.
* Se um passo falha no meio do processo (ex: Pagamento ok, mas Estoque esgotou), não existe "Rollback" no banco do vizinho.
* Em vez disso, usa-se **Ações de Compensação (Compensations)**: Novas ações de negócio que "revertem" o estado logicamente (ex: Fazer um "Estorno" em vez de deletar o registro do pagamento).

---

## 7. O Padrão Outbox (Reliable Publishing)
O problema do **Dual Write**: É comum os desenvolvedores tentarem (1) salvar dados no banco e (2) emitir evento para o broker no mesmo bloco de código. Se o banco salva, mas a rede do broker cai, o sistema fica inconsistente.

**A Solução (Outbox Pattern)**:
1. No mesmo Banco de Dados da sua entidade de negócio, tenha uma tabela `Outbox`.
2. Em **uma mesma transação de banco de dados (ACID)**, você salva as alterações da Entidade E salva uma linha na tabela `Outbox` contendo o payload do Evento.
3. Um processo separado (um Poller ou Change Data Capture - CDC) lê a tabela `Outbox` e envia para o Message Broker de forma segura, marcando como `enviado` após o sucesso.
