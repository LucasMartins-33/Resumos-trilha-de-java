# Questões - Capítulo 10: Testes, Swagger e Rastreabilidade

### Questão 1: Ao testar os endpoints no Postman, que tipo de resposta HTTP indica falha de credencial (Token ausente) e falha de permissão (Roles incorretas)?
<details>
<summary>👀 Ver Resposta</summary>

Status `401 Unauthorized` indica falta de autenticação (Token inválido/ausente). O Status `403 Forbidden` indica que o usuário autenticado não possui o papel (Role) necessário para executar aquela ação.
</details>

### Questão 2: Para solicitar o Token ao Keycloak via Postman, qual aba/método e `grant_type` o instrutor utilizou?
<details>
<summary>👀 Ver Resposta</summary>

Usou a aba *Body* configurada como `x-www-form-urlencoded`, passando as credenciais com o `grant_type=password` para o endpoint `/protocol/openid-connect/token`.
</details>

### Questão 3: Ao deletar uma proposta utilizando o usuário "Manager", por que o item sumiu do `Proposal DB` mas se manteve no `Report DB`?
<details>
<summary>👀 Ver Resposta</summary>

Porque o serviço de Report apenas arquiva as oportunidades recebidas pelo Kafka, mantendo um histórico analítico. O comando DELETE na arquitetura apresentada afeta apenas o banco transacional de Propostas, cumprindo a regra de negócio do cliente.
</details>

### Questão 4: O que é rastreabilidade distribuída (Distributed Tracing)?
<details>
<summary>👀 Ver Resposta</summary>

É uma técnica de monitoramento que acompanha uma requisição HTTP enquanto ela salta de um microsserviço para outro (ex: Frontend -> Gateway -> Backend), identificando o tempo gasto e possíveis gargalos em cada etapa.
</details>

### Questão 5: O que é e para que serve o Jaeger?
<details>
<summary>👀 Ver Resposta</summary>

Jaeger é um sistema open-source de rastreamento distribuído, responsável por coletar e apresentar numa interface visual a rota, o tempo de execução e os possíveis erros entre diferentes microsserviços.
</details>

### Questão 6: No ambiente original do Quarkus (versão 2), como se configurava a rastreabilidade nos *Services* da aplicação?
<details>
<summary>👀 Ver Resposta</summary>

Adicionando a dependência `quarkus-smallrye-opentracing` e anotando as classes de serviço com `@Traced`.
</details>

### Questão 7: Com as atualizações modernas do Quarkus 3+, o que substitui o antigo OpenTracing e a anotação `@Traced`?
<details>
<summary>👀 Ver Resposta</summary>

O novo padrão da indústria é o **OpenTelemetry** (dependência `quarkus-opentelemetry`). A coleta nos endpoints (REST) e CDI ocorre de forma automática sem precisar de `@Traced`. Para mapeamento manual de métodos, usa-se a anotação `@WithSpan`.
</details>

### Questão 8: Qual extensão o Quarkus usa para gerar documentação automática estilo Swagger baseada no código?
<details>
<summary>👀 Ver Resposta</summary>

A extensão `smallrye-openapi`, que expõe os schemas na URL `/q/swagger-ui`.
</details>

### Questão 9: Qual é a dificuldade prática em utilizar o Swagger UI com a segurança Keycloak (`@RolesAllowed`) ativada e qual a dica de desenvolvimento sugerida?
<details>
<summary>👀 Ver Resposta</summary>

O Swagger nativo não enviará o token para endpoints fechados. A dica do instrutor para testes puramente de desenvolvimento é comentar as anotações temporariamente (`@RolesAllowed` e o provider) e religar depois, focando os testes seguros no Postman.
</details>

### Questão 10: No desafio final proposto pelo curso, qual era o objetivo lógico para ser aplicado na classe `QuotationScheduler`?
<details>
<summary>👀 Ver Resposta</summary>

Criar um Scheduler secundário (ex: para rodar no final da noite) que faça a limpeza (`deleteAll()`) do banco da cotação. Isso garantiria que a lógica condicional de aumento de preço do dólar resetasse a cada novo dia útil.
</details>
