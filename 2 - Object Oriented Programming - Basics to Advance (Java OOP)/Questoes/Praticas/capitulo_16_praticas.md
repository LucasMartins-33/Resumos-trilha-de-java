📘 Capítulo 16: SOA, EDA e Microsserviços 

**O Cenário:**
Sua empresa estourou a capacidade do banco de dados Monolítico e agora precisa escalar via Microsserviços de verdade, adotando EDA (Orientação a Eventos).

```java
// O que tínhamos no passado (Monolítico Síncrono)
public void checkout() {
   pagamentoService.pagar();
   estoqueService.baixarItem(); // Se falhar aqui, dá Rollback em tudo.
   emailService.enviarConfirmacao();
}
```

Sua missão é refatorar a mentalidade síncrona para arquitetura distribuída resiliente:

🟢 Atividade 16.1: Quebrando o Banco Compartilhado
1. A regra nº 1 de SOA é Autonomia. O Serviço de Pedido e o de Estoque usavam as mesmas tabelas no MySQL.
2. Defina as fronteiras de serviço (Bounded Contexts) criando dois "projetos/pastas" separados. Cada um terá sua própria String de Conexão.

🟢 Atividade 16.2: Comunicação Assíncrona via Broker
1. Remova as chamadas síncronas de método do seu "checkout" acima.
2. Em vez de chamar o `estoqueService`, o Serviço de Pedidos deverá chamar a infraestrutura de mensageria: `messageBroker.publish(new EventoPedidoCriado(id))`.

🟡 Atividade 16.3: Fatos do Passado (Eventos)
1. Renomeie as ações para seguir o paradigma EDA. "AtualizarEstoque" é um Comando. O nome do Evento DEVE estar no passado.
2. Crie a classe (record) `EstoqueAtualizadoEvent`. Ele é um Fato consumado e inegável!

🟡 Atividade 16.4: Consumidor Idempotente (At-least-once)
1. O Apache Kafka caiu e, ao voltar, enviou o `PedidoCriadoEvent (id=99)` duas vezes!
2. Na classe do seu `EstoqueConsumer`, crie a lógica em pseudo-código: Antes de baixar o estoque, faça um `SELECT` para verificar se o `pedido_id 99` já não foi processado antes. Se sim, ignore silenciosamente. (Isso é ser idempotente).

🟠 Atividade 16.5: Tradução de Eventos (Domain vs Integration)
1. Seu serviço gera o `DomainEvent`: `PedidoCriado (com 50 campos sensíveis, como CpfCliente)`.
2. Crie o `IntegrationEvent`: `PublicPedidoCriado (apenas IdPedido e ValorTotal)`.
3. Crie a camada de tradução antes de jogar o evento na Fila Pública para o mundo externo ler.

🟠 Atividade 16.6: O Padrão SAGA (Transação Local 1)
O banco central morreu, não há mais `COMMIT` global. Faremos uma SAGA.
1. O Pedido cria no próprio banco o status `PROCESSANDO_PAGAMENTO`.
2. Publica o evento.
3. O Serviço de Pagamentos recebe, debita no cartão com sucesso, atualiza o próprio banco e publica `PagamentoAprovado`.

🔴 Atividade 16.7: Ações de Compensação (Falha na SAGA)
O Pagamento deu certo, mas o Estoque falhou (Item Indisponível).
1. O Estoque publica `EstoqueRejeitado`.
2. Escreva o fluxo onde o Serviço de Pagamento escuta esse evento e executa a "Compensação": um Estorno no cartão de crédito! (Nunca tentamos fazer "Rollback" distribuído).

🔴 Atividade 16.8: Coreografia
1. Em vez de ter um Maestro ditando quem deve fazer o quê, deixe cada serviço apenas escutar e reagir.
2. Qual o risco prático a longo prazo dessa abordagem (Dica: Rastreabilidade)?

🔴 Atividade 16.9: O Problema do Dual Write
```java
// Anti-padrão comum!
db.salvarPedido(pedido);
kafka.enviar(eventoPedidoCriado); // O que acontece se a luz acabar EXATAMENTE nesta linha?
```
1. Identifique e explique o que acontece com a consistência de dados dos outros serviços caso o cenário comentado acima aconteça.

🔴 Atividade 16.10: O Padrão Outbox (A Salvação)
1. Conserte o Dual Write acima.
2. Inicie a transação ACID local.
3. Insira o Pedido na tabela `pedidos`.
4. Insira os dados (JSON) do Evento na tabela `outbox`.
5. Faça o `commit` final no banco. Se a luz acabar, ou não salva nada, ou salva ambos! Um serviço em background lerá a tabela outbox enviando para o Kafka assincronamente.
