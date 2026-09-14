# Capítulo 02: Entendendo o Desafio e os Requisitos da Aplicação Web

Este documento contém o resumo do **Capítulo 02**, focado no entendimento do projeto prático do curso: uma **Proof of Concept (POC)** para a empresa fictícia "BR Mineradora", envolvendo o levantamento de requisitos funcionais, técnicos e a proposta de arquitetura baseada em microserviços.

---

## 1. O Desafio e Regra de Negócio (Requisitos Funcionais)
A **BR Mineradora** exporta minério de ferro, negociando em **Dólar americano (USD)**. Portanto, o lucro da empresa é influenciado pela cotação do Dólar em relação ao Real (BRL).
O objetivo é desenvolver uma API que cruze dados de propostas de clientes com a cotação atual do dólar para gerar relatórios de oportunidades de venda.

### Funcionalidades Esperadas:
1. **Acompanhamento de Cotação:** A aplicação deve consultar periodicamente o valor do dólar e registrar no banco de dados.
2. **Entrada de Propostas:** Recebimento de novas propostas de compra contendo dados como empresa, valor oferecido, quantidade de toneladas, país e validade.
3. **Geração de Oportunidades:** Cruzamento das propostas com a cotação do dólar, gerando relatórios em formato **JSON** ou **CSV**.

### Regras de Acesso e Perfis de Usuário:
Para garantir a segurança, o sistema deverá ter controle de acesso baseado em perfis (Roles):
- **Cliente:** O único perfil autorizado a **inserir** novas propostas no sistema.
- **Operador (de negócios):** Pode apenas **consultar** as propostas e extrair relatórios (não pode inserir nem deletar).
- **Gerente:** Pode **consultar** detalhes e também **deletar** propostas.

---

## 2. Requisitos Técnicos
A equipe técnica da mineradora impôs alguns direcionamentos que a nossa POC precisa atender:
- **Cloud First:** A aplicação vai rodar na nuvem, logo precisa ser leve para evitar desperdício de infraestrutura.
- **Ecossistema Java:** A empresa já possui um background forte em Java (antigo Java EE) e exige que as novas aplicações mantenham a linguagem, aproveitando os conhecimentos da equipe.
- **Redução de Custos (CPU/RAM):** Embora Java tradicional consuma muito recurso (o que encarece a nuvem), a arquitetura deve provar que é possível rodar Java com baixo custo (é aqui que o **Quarkus** entra).
- **Resiliência e Alta Disponibilidade:** A aplicação precisa funcionar 24 horas por dia. Se uma parte do sistema cair, as outras devem continuar operando (justificativa para o uso de Microserviços e Mensageria).

---

## 3. Arquitetura Proposta
Para atender aos requisitos, a arquitetura foi desenhada de forma **híbrida** (comunicação REST + Mensageria Kafka):

### Componentes de Entrada e Segurança:
- **Keycloak:** Servidor de identidade responsável por validar credenciais (usuário e senha) e devolver um token de autorização.
- **JWT (JSON Web Token):** O token que trafega entre os serviços carregando a identidade do usuário e suas permissões (se é Cliente, Operador ou Gerente).
- **BFF (Back-End For Front-End) / API Gateway:** O único ponto de entrada que recebe requisições externas (ex: do Postman). Ele checa a validade do JWT e redireciona a chamada para o microserviço adequado, mascarando as APIs internas.

### Os Microserviços de Negócio:
Cada microserviço é construído em Quarkus e possui **seu próprio banco de dados**.
1. **Microserviço de Cotação:** 
   - Consome uma API REST *externa* para pegar o valor atual do Dólar.
   - Salva no banco e publica a cotação em um tópico do **Apache Kafka**.
   - *Nota:* Não possui endpoints REST expostos (não recebe chamadas do Gateway).
2. **Microserviço de Proposta:** 
   - Chamado pelo Gateway quando um "Cliente" envia uma nova proposta.
   - Valida a regra, salva no seu banco e envia os dados para um tópico do **Apache Kafka**.
3. **Microserviço de Report (Relatórios):** 
   - Funciona como um consumidor do Kafka: lê as novas cotações e propostas conforme chegam nos tópicos.
   - Processa o cruzamento dessas informações e salva as oportunidades no seu banco de dados.
   - Expõe endpoints REST (JSON e CSV) que são acessados por "Operadores" e "Gerentes" através do Gateway.

### Observabilidade (Tracing):
- **Jaeger:** Ferramenta de rastreamento (Distributed Tracing). Como uma requisição pode passar pelo Gateway, depois para o serviço de Proposta e etc., o Jaeger ajuda a visualizar o caminho completo da requisição, facilitando a identificação de gargalos de performance e bugs na comunicação entre os microserviços.

---
*Resumo: A separação das responsabilidades e o uso do Apache Kafka garantem que, se o serviço de Propostas cair, o serviço de Relatórios continue permitindo que os operadores leiam as oportunidades geradas anteriormente, provando a resiliência exigida pelo cliente.*
