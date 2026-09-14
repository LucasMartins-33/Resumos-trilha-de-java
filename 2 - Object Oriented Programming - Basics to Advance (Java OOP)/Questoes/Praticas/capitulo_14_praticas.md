📘 Capítulo 14: Modelando Comportamento (Diagramas Comportamentais)

**O Cenário:**
O sistema está pronto, mas as regras de negócio são obscuras. Os devs júnior precisam de diagramas de comportamento para entender fluxos.

Você modelará a parte dinâmica usando descrições de passos UML:

🟢 Atividade 14.1: Casos de Uso Base
1. Defina um Ator Principal: "Cliente".
2. Defina 2 casos de uso ovais: "Fazer Pedido" e "Acompanhar Rastreio".
3. Conecte o Ator com linhas simples até os casos de uso.

🟢 Atividade 14.2: Include no Caso de Uso
1. Identifique um comportamento que SEMPRE ocorre no fluxo de "Fazer Pedido": "Validar Cartão de Crédito".
2. Desenhe uma seta tracejada apontando para o caso secundário com a anotação `<<include>>`.

🟡 Atividade 14.3: Extend no Caso de Uso
1. Identifique um comportamento opcional ao fazer pedido: "Aplicar Cupom de Desconto".
2. Como ele é condicional (opcional), crie a seta com a anotação `<<extend>>`. Mas atenção: a direção da seta do extend normalmente aponta PARA o caso principal!

🟡 Atividade 14.4: Início de Diagrama de Sequência
1. Crie 3 Linhas de Vida (Lifelines): `ClientApp`, `ApiServer`, `Database`.
2. Desenhe a mensagem Síncrona 1: de `ClientApp` para `ApiServer` chamando `login()`.

🟠 Atividade 14.5: Retorno e Assincronismo (Sequência)
1. Desenhe a linha tracejada de retorno com os dados `Token JWT` do Servidor para o Client.
2. O Servidor também dispara um email assíncrono para notificar o login. Como a ponta da seta muda (aberta vs preenchida) no diagrama de Sequência?

🟠 Atividade 14.6: Blocos Condicionais (Combined Fragment - Alt)
1. No login, a senha pode estar errada. Desenhe o bloco Retangular na Lifeline do Servidor rotulado como `alt`.
2. Adicione a condição `[senha correta]` e a condição `[senha incorreta]` dividindo o bloco em duas metades com comportamentos diferentes.

🔴 Atividade 14.7: Diagrama de Atividades (Fluxograma Avançado)
1. Comece com um Círculo Sólido Preto (Nó Inicial).
2. Coloque um losango de decisão: "Usuário é Admin?".
3. Se Sim, vai para a Atividade "Deletar Produto". Se não, vai para o "Nó Final" (Alvo).

🔴 Atividade 14.8: Concorrência com Forks (Atividades)
1. O usuário finalizou o pedido. Três coisas devem acontecer ao mesmo tempo.
2. Desenhe um `Fork` (Barra grossa preta).
3. A partir dele, lance 3 setas de fluxo paralelas: "Mandar Email", "Gerar NF", "Separar Estoque". Junte-as no final com um `Join`.

🔴 Atividade 14.9: Máquina de Estados (Estado Simples)
1. Modele o ciclo de vida da classe `Pedido`. O nó inicial vai para o estado `PENDENTE_PAGAMENTO`.
2. Defina uma Transição: do estado atual para `PROCESSANDO`, engatilhada pelo Evento `CartaoAprovado`.

🔴 Atividade 14.10: Transições e Guards (Máquina de Estados)
1. E se o cartão falhar? Crie a transição de `PENDENTE_PAGAMENTO` para `CANCELADO`.
2. Adicione um *Guard* na transição: `[tentativas > 3]`. Ou seja, o estado só muda para cancelado após três falhas.
