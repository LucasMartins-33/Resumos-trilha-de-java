# Capítulo 10: Testes Finais, Jaeger (Rastreabilidade) e Swagger UI

Este documento resume o **Capítulo 10**, onde consolidamos todo o ecossistema construído colocando todos os serviços no ar simultaneamente. Testamos as regras de negócio via Postman, implementamos o rastreamento de requisições com Jaeger e visualizamos a documentação das APIs com o Swagger.

---

## 1. Testes End-to-End com Postman
O instrutor subiu simultaneamente: os 4 microsserviços (Cotação, Proposta, Report e Gateway), os bancos de dados, o Kafka e o Keycloak.
Todas as requisições (a partir de agora) passaram a ser enviadas apenas para a porta do **Gateway (`8095`)**, simulando o mundo real.

### Fluxo de Autenticação
Para consumir qualquer API do Gateway, foi necessário primeiro obter o *Token JWT*:
1. Rota do Keycloak: `POST http://localhost:8180/realms/quarkus/protocol/openid-connect/token`
2. Parâmetros (x-www-form-urlencoded): `grant_type=password`, `username`, `password`, `client_id=backend-service`, `client_secret=secret`.
3. O token retornado é copiado e enviado na aba *Authorization (Bearer Token)* das requisições do Gateway.

### Validação das Regras de Negócio (RBAC)
- **Bloqueio Correto (403 Forbidden):** Tentar criar uma proposta logado como o funcionário "João" (`user`) resultou em acesso proibido.
- **Sucesso (200 OK):** Logar com o cliente "China Miner" (`proposal-customer`) permitiu criar a proposta. O dado fluiu perfeitamente do Gateway -> Proposta -> Kafka -> Report.
- **Proteção de Fraude:** Tentar ler o relatório geral das propostas de todos os concorrentes utilizando o token do cliente "China Miner" foi bloqueado (403), garantindo a confidencialidade da mineradora.
- **Exclusão Isolada:** Apenas o gerente "José" (`manager`) conseguiu deletar propostas. Detalhe arquitetural: a deleção remove os dados do banco de *Propostas*, mas **mantém** o registro histórico no banco de *Reports* (comportamento definido pela regra de negócio, não é um erro).

---

## 2. Jaeger, OpenTracing e OpenTelemetry (Rastreabilidade Distribuída)
Em uma arquitetura com vários microsserviços, se uma requisição falha, é muito difícil saber em qual serviço ocorreu o erro. Para isso, implementou-se a rastreabilidade usando **Jaeger**.

### Implementação Original (Conforme a aula)
No vídeo, o instrutor utiliza a especificação OpenTracing:
1. **Dependência:** Adicionado o `quarkus-smallrye-opentracing` no `pom.xml` dos serviços de Gateway, Proposta e Report.
2. **Anotação:** A anotação `@Traced` foi colocada em cima das classes *Service* (os *Controllers* são rastreados automaticamente).
3. **Execução:** O servidor do Jaeger foi executado via container Docker (acesso visual na porta `16686`).

### ⚠️ Atualização Importante (Quarkus 3.x e OpenTelemetry)
O `quarkus-smallrye-opentracing` e a anotação `@Traced` foram **descontinuados/removidos** nas versões mais recentes do Quarkus (3+), sendo substituídos pelo padrão moderno da indústria, o **OpenTelemetry**.
Se você estiver codificando em uma versão recente:
- **Dependência atualizada:** Você deve usar a extensão `quarkus-opentelemetry`.
- **Fim do `@Traced`:** Com o OpenTelemetry, o rastreamento das requisições REST (JAX-RS) e injeções de dependência ocorre automaticamente, dispensando a necessidade do `@Traced` em cada Service. 
- **Spans Customizados:** Caso precise monitorar métodos específicos que não estão mapeados automaticamente, a anotação recomendada atualmente é o **`@WithSpan`** (`io.opentelemetry.instrumentation.annotations.WithSpan`).
- **Comunicação:** O Quarkus passa a enviar os rastros via protocolo OTLP (normalmente para a porta `4317`), e a imagem Docker do Jaeger já aceita nativamente esse formato.

### Resultado
- Ao fazer uma requisição no Gateway, a interface gráfica do Jaeger mostrou o "caminho" (Span) exato do dado: `Gateway Controller -> Gateway Service -> Report Controller -> Report Service`.
- O instrutor simulou uma queda: derrubou intencionalmente o microsserviço de Report. Ao disparar no Postman, a interface do Jaeger plotou exatamente a linha vermelha informando que a conexão na porta `8081` foi recusada, poupando o desenvolvedor de caçar erros nos logs.

---

## 3. OpenAPI e Swagger UI
A dependência `smallrye-openapi` gera automaticamente uma documentação interativa baseada no código.
- **Acesso:** `http://localhost:8095/q/swagger-ui`
- O painel exibe e documenta perfeitamente as rotas `/api/trade` e `/api/opportunity`.
- **Dica de Desenvolvimento:** Como os endpoints estão totalmente trancados pelo Keycloak (`@RolesAllowed`), fazer testes rápidos via Swagger exige configurações complexas de OIDC no Swagger UI. Para ambiente de desenvolvimento local, o instrutor mostrou que basta comentar temporariamente as anotações de segurança (e o *RegisterProvider* do token) para brincar livremente no Swagger.

---

## 4. Desafio Final (Melhoria do Scheduler)
O instrutor deixou um desafio prático de código:
Na lógica atual, o microsserviço de **Cotação** só envia atualizações para o Kafka se o valor do dólar **aumentar** em relação à última leitura salva. Se o dólar cair de um dia para o outro, o sistema trava no valor antigo mais alto.
- **A solução proposta:** Criar um novo método `@Scheduled` para rodar no final de cada dia (ex: 23:59). Esse método deve disparar um `Repository.deleteAll()` limpando o banco de cotações, permitindo que a lógica recomece limpa na manhã seguinte.
