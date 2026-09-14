📘 Capítulo 04: Desenvolvimento Microserviço de Proposta

O Cenário:
O Microserviço de Proposta é a porta de entrada para os clientes enviarem novos dados de compra. Ele armazena as propostas detalhadas e envia uma versão enxuta para o Apache Kafka.

Sua missão é testar seus conhecimentos sobre o padrão MVC, persistência e emissão de eventos dentro desse microserviço:

🟢 Atividade 4.1: A Estrutura de Pacotes (Clean Architecture base)

O projeto dividiu suas responsabilidades em pacotes como `controller`, `service`, `repository`, etc.
Qual pacote deve ser o único responsável por validar as regras de negócio?

🟢 Atividade 4.2: DTOs e Transferência Otimizada

Foram criados dois DTOs: `ProposalDetailsDTO` e `ProposalDTO`.
Por que o objeto enviado ao Kafka (`ProposalDTO`) contém menos atributos do que o objeto de detalhes? Qual é a vantagem de enviar apenas os campos essenciais?

🟢 Atividade 4.3: Custom Queries no PanacheRepository

O Panache traz vários métodos prontos, mas às vezes precisamos criar consultas personalizadas.
Escreva a linha de código, usando o método `find()`, para buscar uma proposta específica baseada na coluna `customer` no banco de dados.

🟢 Atividade 4.4: Injeção de Dependências e o Escopo de Aplicação

O Quarkus utiliza Injeção de Dependência. 
Na implementação `ProposalServiceImpl`, qual anotação é obrigatoriamente necessária no nível da classe para evitar o erro "Unsatisfied dependency" ao subir a aplicação?

🟢 Atividade 4.5: Protegendo Alterações no Banco

Os métodos de criar e remover propostas modificam o banco de dados.
Qual anotação deve ser colocada acima desses métodos no nível do *Service* para garantir a integridade da operação?

🟢 Atividade 4.6: Exposição da API (Controller)

A classe `ProposalController` precisa responder na rota `/api/proposal`.
Qual anotação do JAX-RS define essa URL base na classe?

🟢 Atividade 4.7: Boas Práticas do MVC em APIs

A requisição HTTP chega através do *Controller*.
Por que é considerada uma má prática acessar o *Repository* (banco de dados) diretamente no Controller, ao invés de delegar para o *Service*?

🟢 Atividade 4.8: Envio de Mensagem para o Kafka

Sempre que uma nova proposta é criada, o serviço precisa avisar os outros.
Qual ferramenta específica o instrutor utilizou (dependência) para simplificar essa integração reativa com o Kafka no Quarkus?

🟢 Atividade 4.9: Configuração de Múltiplos Bancos de Dados

O Microserviço de Proposta usa o banco `proposal_db` enquanto o de Cotação usa `quotation_db`.
Qual é a principal justificativa arquitetural de cada microserviço ter seu próprio banco de dados independente?

🟢 Atividade 4.10: Teste da API via Postman

Ao enviar uma requisição `POST` pelo Postman para testar a criação de proposta.
Quais informações precisam ser enviadas obrigatoriamente no corpo (Body) da requisição (no formato JSON) para ela ser bem-sucedida de acordo com a entidade criada?
