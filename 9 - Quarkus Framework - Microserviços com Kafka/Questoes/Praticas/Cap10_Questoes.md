📘 Capítulo 10: Testes Finais, Jaeger (Rastreabilidade) e Swagger UI

O Cenário:
O sistema está no ar! Os 4 microsserviços, Kafka, Banco de Dados e Keycloak estão rodando em paralelo. Você precisa gerar os tokens e testar todo o ecossistema na prática, além de analisar falhas distribuídas usando o Jaeger.

Sua missão é testar seus conhecimentos práticos sobre integração end-to-end e observabilidade no Quarkus moderno:

🟢 Atividade 10.1: Obtendo o Token JWT (Autenticação via API)

Para testar no Postman, você não tem uma tela de login.
Qual requisição (Método HTTP + Rota do Keycloak) deve ser feita, e quais são os parâmetros essenciais (`grant_type`, `client_id`, `username`, `password`) que devem ir no corpo (x-www-form-urlencoded) para receber o Token JWT?

🟢 Atividade 10.2: Autenticação via Header HTTP

Uma vez obtido o token gigantesco do Keycloak.
Qual é o nome exato do cabeçalho HTTP (Header) que o Postman deve enviar para o Gateway, e qual palavra-chave (prefixo) deve acompanhar o token? (Dica: `____: _____ xyz123Token`).

🟢 Atividade 10.3: Teste de Bloqueio Efetivo (403 Forbidden)

Se você estiver logado com o JWT do cliente "China Miner" (que possui a role `proposal-customer`) e tentar fazer uma requisição `GET` no Gateway para baixar relatórios de oportunidades (que exige `user`).
Qual erro HTTP será retornado e por que o código de erro difere do Erro `401 Unauthorized`?

🟢 Atividade 10.4: O que é Distributed Tracing?

Em uma arquitetura de microserviços, as requisições saltam de uma aplicação para outra.
Explique o conceito de "Distributed Tracing" e qual é a principal dor que ferramentas como o Jaeger resolvem no dia a dia do suporte técnico.

🟢 Atividade 10.5: O fim do @Traced e do OpenTracing

Nas versões modernas do Quarkus (3.x), ocorreu uma evolução na rastreabilidade.
Qual tecnologia substituiu o OpenTracing (e a dependência `smallrye-opentracing`) tornando-se o padrão nativo da indústria para observabilidade no Quarkus moderno?

🟢 Atividade 10.6: Telemetria Automática

Usando o OpenTelemetry no Quarkus, nós precisamos colocar anotações específicas para que as requisições HTTP entre o Gateway e o Report apareçam no painel do Jaeger? Explique o comportamento padrão da biblioteca atual.

🟢 Atividade 10.7: Acesso Visual do Jaeger

Após configurar o Quarkus para exportar os logs (usualmente via protocolo OTLP).
Na simulação do instrutor, o erro de conexão recusada (connection refused) quando o Report estava offline apareceu grifado no painel. Como isso impacta a velocidade do diagnóstico de incidentes em produção?

🟢 Atividade 10.8: A Mágica do Swagger UI

Ao usar o `smallrye-openapi`, o Quarkus expõe a documentação baseada nas especificações da OpenAPI.
Qual a URL padrão de desenvolvimento no Quarkus onde você acessa a interface visual do Swagger para poder enviar testes de requisição via navegador?

🟢 Atividade 10.9: O Desafio da Cotação Diária

No fim do curso, o instrutor deixou o desafio de resetar as cotações à meia-noite, pois o código base trava no "maior dólar já registrado".
Se você fosse criar a lógica no `QuotationScheduler`, como seria a cron expression no Quarkus `@Scheduled` para rodar todos os dias às `23:59:59`? E qual método do repositório deve ser chamado para limpar a tabela?

🟢 Atividade 10.10: Deleção Isolada por Microserviços

O teste mostrou que deletar uma proposta removeu do DB de propostas, mas não do DB de Relatórios (Report).
Em arquitetura orientada a eventos (Kafka), como poderíamos notificar o microserviço de Report de que uma proposta foi deletada para que ele a apagasse do seu banco também (caso fosse uma regra de negócio solicitada pelo cliente)?
