# Questões - Capítulo 01: Introdução

### Questão 1: Qual é o principal objetivo arquitetural do curso em relação à comunicação entre microsserviços?
<details>
<summary>👀 Ver Resposta</summary>

O curso foca em uma abordagem híbrida, utilizando comunicação síncrona via APIs REST (com um API Gateway/BFF) e comunicação assíncrona orientada a eventos utilizando o Apache Kafka como broker de mensagens.
</details>

### Questão 2: O que é o Apache Kafka e qual seu papel na arquitetura?
<details>
<summary>👀 Ver Resposta</summary>

O Apache Kafka é uma plataforma distribuída de mensageria e streaming de eventos. Na arquitetura do curso, ele atua como o intermediário (broker) que recebe eventos de um microsserviço (Producer) e os entrega para outros (Consumers), garantindo desacoplamento e resiliência.
</details>

### Questão 3: Por que utilizar o padrão de eventos (mensageria) em vez de apenas chamadas REST diretas entre todos os microsserviços?
<details>
<summary>👀 Ver Resposta</summary>

Porque o padrão de eventos reduz o acoplamento temporal. Se o serviço de destino estiver fora do ar, a mensagem fica guardada no Kafka e será processada assim que o serviço voltar, evitando a perda de dados e falhas em cascata que ocorreriam numa chamada REST síncrona.
</details>

### Questão 4: O que é o Quarkus e qual a sua principal vantagem?
<details>
<summary>👀 Ver Resposta</summary>

Quarkus é um framework Java nativo para Kubernetes, desenhado para ter um tempo de inicialização (startup) extremamente rápido e um baixo consumo de memória, características ideais para arquiteturas de microsserviços e serverless.
</details>

### Questão 5: O que é o Keycloak mencionado na introdução do curso?
<details>
<summary>👀 Ver Resposta</summary>

O Keycloak é um servidor Open Source de Gerenciamento de Identidade e Acesso (IAM). Ele será usado para proteger os microsserviços, emitindo tokens JWT para usuários autenticados.
</details>

### Questão 6: Qual o papel de um API Gateway ou BFF (Backend For Frontend) na arquitetura apresentada?
<details>
<summary>👀 Ver Resposta</summary>

Ele serve como a única porta de entrada pública para os clientes externos. Ele recebe as requisições, verifica a segurança e as roteia para os microsserviços internos corretos, podendo também formatar os dados especificamente para o frontend que os solicitou.
</details>

### Questão 7: Em uma arquitetura de microsserviços, como os bancos de dados devem ser tratados?
<details>
<summary>👀 Ver Resposta</summary>

Cada microsserviço deve ter o seu próprio banco de dados isolado. Compartilhar um único banco de dados gigante entre vários microsserviços quebra o princípio de independência e cria um alto acoplamento (anti-pattern conhecido como Shared Database).
</details>

### Questão 8: O que significa dizer que a comunicação via Kafka é assíncrona?
<details>
<summary>👀 Ver Resposta</summary>

Significa que o microsserviço que envia a mensagem (Producer) não fica esperando uma resposta do microsserviço que vai receber a mensagem (Consumer). Ele apenas posta o evento no tópico e continua seu trabalho.
</details>

### Questão 9: Quais são os 3 principais microsserviços de negócio que serão construídos ao longo do curso?
<details>
<summary>👀 Ver Resposta</summary>

1. Cotação (busca preços do dólar); 2. Proposta (recebe ofertas de compra dos clientes); 3. Report/Relatório (consolida os dados de cotações e propostas para análise).
</details>

### Questão 10: Qual a linguagem e a versão principal utilizada ao longo do projeto (e qual a atualização recomendada)?
<details>
<summary>👀 Ver Resposta</summary>

O curso foi gravado utilizando Java 11. No entanto, para projetos modernos com Quarkus 3+, a versão recomendada para implementação hoje é o Java 17 ou superior.
</details>
