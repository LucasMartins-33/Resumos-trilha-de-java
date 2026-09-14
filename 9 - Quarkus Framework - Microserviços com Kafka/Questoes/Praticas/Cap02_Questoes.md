📘 Capítulo 02: Entendendo o Desafio e os Requisitos da Aplicação Web

O Cenário:
Você foi designado para arquitetar uma POC (Proof of Concept) para a "BR Mineradora". A empresa precisa de um sistema resiliente na nuvem para analisar propostas de clientes com base na cotação atual do dólar.

Sua missão é entender as regras de negócio, os perfis de acesso e a arquitetura em microserviços proposta:

🟢 Atividade 2.1: Identificando as Funcionalidades Core

A POC possui 3 grandes responsabilidades funcionais.
Liste quais são elas e identifique quais microserviços assumem cada uma dessas responsabilidades.

🟢 Atividade 2.2: Mapeamento de Perfis de Usuário (Roles)

A aplicação requer controle de acesso estrito.
Escreva qual o nível de acesso (Inserir, Consultar, Deletar) para cada um dos seguintes perfis: Cliente, Operador e Gerente.

🟢 Atividade 2.3: Requisitos Cloud First e Custos

O cliente exige que a aplicação rode na nuvem de forma barata, mas insiste no ecossistema Java.
Como a escolha do framework (Quarkus) justifica o atendimento desses dois requisitos (Java + Baixo custo de CPU/RAM)?

🟢 Atividade 2.4: O Papel do Gateway / BFF

Na arquitetura proposta, existe um componente chamado API Gateway / BFF (Back-End For Front-End).
Qual é o objetivo principal desse Gateway em relação às requisições externas e aos microserviços internos?

🟢 Atividade 2.5: Segurança com Keycloak e JWT

O Gateway não valida os usuários sozinho; ele trabalha em conjunto com o Keycloak.
Explique o fluxo básico: como um usuário recebe o JWT e como o Gateway utiliza esse JWT para permitir o acesso.

🟢 Atividade 2.6: Microserviço de Cotação - Isolamento

O Microserviço de Cotação consome a API externa do Dólar e posta num tópico do Kafka, mas ele **não possui endpoints REST expostos**.
Por que essa decisão arquitetural é considerada segura? Como os outros serviços ficam sabendo da cotação sem acessar o banco de dados dele?

🟢 Atividade 2.7: Microserviço de Proposta - Fluxo de Dados

Um Cliente envia uma proposta.
Descreva o passo a passo (fluxo) desde a chamada no Gateway, a validação no microserviço, o armazenamento em banco, até o envio da mensagem para o Apache Kafka.

🟢 Atividade 2.8: Microserviço de Report - Consumo Assíncrono

O Microserviço de Report precisa gerar relatórios cruzando Propostas e Cotações, mas ele não tem acesso ao banco de dados dos outros serviços.
Como ele mantém seus próprios dados atualizados e por que ele atua como um "Consumidor" nessa arquitetura?

🟢 Atividade 2.9: Resiliência da Arquitetura

Imagine que o Microserviço de Propostas caia (fique offline).
O que acontece com o Microserviço de Relatórios? Ele ainda consegue gerar relatórios das propostas anteriores? Justifique com base na arquitetura.

🟢 Atividade 2.10: Observabilidade com Jaeger

Em um ambiente com múltiplos microserviços e Gateway, rastrear erros é complexo.
Qual é a finalidade de usar a ferramenta Jaeger (Distributed Tracing) nesse ecossistema?
