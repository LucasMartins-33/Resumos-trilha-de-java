# Questões - Capítulo 02: Entendendo o Desafio

### Questão 1: O que significa a sigla POC e qual o seu propósito?
<details>
<summary>👀 Ver Resposta</summary>

POC significa "Proof of Concept" (Prova de Conceito). É um software inicial e funcional criado para demonstrar a viabilidade técnica de uma solução ou arquitetura antes de investir no desenvolvimento do produto final para produção.
</details>

### Questão 2: Qual é o cenário de negócio da empresa fictícia "BR Mineradora"?
<details>
<summary>👀 Ver Resposta</summary>

A BR Mineradora vende minério de ferro para clientes internacionais e precisa de uma plataforma onde os clientes possam enviar propostas de compra, enquanto a empresa analisa essas propostas cruzando-as com a cotação do dólar em tempo real.
</details>

### Questão 3: Por que o microsserviço de "Cotação" não deve expor endpoints REST para a internet?
<details>
<summary>👀 Ver Resposta</summary>

Porque sua única responsabilidade é consultar uma API externa (AwesomeAPI) e avisar o restante do sistema (via Kafka) sobre o aumento do dólar. Ele não tem interação direta com o usuário final, portanto mantê-lo fechado aumenta a segurança.
</details>

### Questão 4: Quais dados essenciais compõem uma "Proposta" de compra na plataforma?
<details>
<summary>👀 Ver Resposta</summary>

A proposta (enviada pelo cliente) deve conter quem é o cliente, o país de destino, a quantidade de minério em toneladas, o preço oferecido por tonelada e a validade da proposta em dias.
</details>

### Questão 5: Como o microsserviço de Relatórios (Report) fica sabendo das novas propostas se não há uma chamada REST direta para ele?
<details>
<summary>👀 Ver Resposta</summary>

Através do Apache Kafka. Quando uma nova proposta é criada no microsserviço de Proposta, um evento é publicado num tópico do Kafka. O microsserviço de Relatórios "assina" (consome) esse tópico e é notificado automaticamente.
</details>

### Questão 6: Por que a arquitetura planeja um banco de dados independente para o microsserviço de Relatórios?
<details>
<summary>👀 Ver Resposta</summary>

Para que o serviço de Relatórios possa fazer agregações e consultas pesadas sem impactar a performance do banco de dados transacional do serviço de Propostas (CQRS/Isolamento de recursos).
</details>

### Questão 7: Qual é o formato do token de segurança que será utilizado pelo Keycloak para validar usuários na arquitetura?
<details>
<summary>👀 Ver Resposta</summary>

JSON Web Token (JWT). O token carrega a identidade do usuário, suas permissões e a validade da autenticação.
</details>

### Questão 8: Qual é o perigo de se usar a comunicação síncrona (REST) entre os microsserviços de negócio (Proposta e Relatório)?
<details>
<summary>👀 Ver Resposta</summary>

Se o microsserviço de Relatório cair, a chamada REST falharia, e o microsserviço de Proposta também não conseguiria completar sua operação de salvar a proposta, gerando uma indisponibilidade em cadeia.
</details>

### Questão 9: Em termos de segurança, qual é a diferença entre os "Operadores de Negócio" e os "Clientes" no sistema da BR Mineradora?
<details>
<summary>👀 Ver Resposta</summary>

Clientes só podem ter acesso à funcionalidade de enviar (criar) novas propostas. Já os Operadores de Negócio (funcionários da BR Mineradora) têm permissão para acessar os relatórios contendo todas as propostas de todos os clientes.
</details>

### Questão 10: Na arquitetura proposta, quem será o responsável por compilar e formatar o relatório final em arquivo CSV?
<details>
<summary>👀 Ver Resposta</summary>

O API Gateway (BFF - Backend For Frontend). O microsserviço de Relatório fornece apenas os dados em JSON, e o Gateway formata esses dados como um arquivo CSV caso o frontend solicite esse formato.
</details>
