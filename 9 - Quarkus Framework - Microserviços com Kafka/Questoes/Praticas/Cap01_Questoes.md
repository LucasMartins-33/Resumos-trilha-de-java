📘 Capítulo 01: Introdução ao Quarkus, Microserviços e Kafka

O Cenário:
O ecossistema Java moderno migrou de monolitos pesados para microserviços leves rodando em containers. O Quarkus, junto com o Apache Kafka, é a base dessa nova arquitetura.

Sua missão é exercitar e validar os conceitos fundamentais sobre arquitetura, comunicação REST e mensageria que formam a base deste projeto:

🟢 Atividade 1.1: Identificando Monolitos vs Microserviços

Descreva em um documento ou bloco de notas a diferença principal entre uma arquitetura de Monolito e uma de Microserviços.
Identifique o que é um SPOF (Ponto Único de Falha) no contexto de um Monolito.
Explique como os microserviços resolvem o problema de resiliência.

🟢 Atividade 1.2: Entendendo o papel do Apache Kafka

O Kafka atua como intermediário de comunicação. Crie um diagrama simples (pode ser em texto) mostrando a relação entre: Produtores (Producers), Tópicos (Topics) e Consumidores (Consumers).
Explique por que a comunicação através do Kafka evita o acoplamento e gargalos em comparação a chamadas HTTP diretas.

🟢 Atividade 1.3: Retenção de Mensagens no Kafka

Investigue a propriedade de retenção de mensagens em um tópico do Kafka.
Se uma mensagem é lida por um Consumidor, ela é apagada do Kafka imediatamente? Responda com base no comportamento padrão de 7 dias.

🟢 Atividade 1.4: KRaft vs Zookeeper

Nas versões mais recentes do Kafka, o Zookeeper foi descontinuado em favor do KRaft.
Pesquise e descreva a principal vantagem do KRaft sobre o Zookeeper na arquitetura do Kafka.

🟢 Atividade 1.5: Verbos HTTP em APIs REST

Dado um sistema REST, mapeie qual verbo HTTP (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`) deve ser usado para:
1) Criar um novo usuário.
2) Buscar uma lista de usuários.
3) Atualizar apenas o telefone de um usuário.
4) Deletar um usuário.

🟢 Atividade 1.6: A vantagem do Build Time no Quarkus

O Quarkus move grande parte do processamento para o tempo de compilação (Build Time).
Liste os principais benefícios práticos dessa abordagem quando comparado ao Java tradicional.

🟢 Atividade 1.7: Compilação AOT (Ahead-Of-Time) e GraalVM

Explique o que é a compilação AOT permitida pela GraalVM.
Qual a relação entre GraalVM e o baixo consumo de RAM (memória "subatômica") das aplicações Quarkus?

🟢 Atividade 1.8: Executando Comandos de Build Nativo

Escreva o comando Maven necessário para gerar um binário nativo no Quarkus usando o GraalVM.
Se você não tem o GraalVM instalado na sua máquina, qual flag você pode adicionar a esse comando para delegar o build para o Docker?

🟢 Atividade 1.9: Inicialização do Executável Nativo

Após o build nativo, o executável é gerado na pasta `target`.
Qual comando você utilizaria no terminal Linux/Mac para rodar esse arquivo, sem utilizar o comando tradicional `java -jar`?

🟢 Atividade 1.10: Requisitos de Versão do Java no Quarkus 3+

Embora o curso antigo possa ter usado Java 11, o Quarkus evoluiu.
Qual é a versão mínima do Java exigida para rodar o Quarkus nas versões atuais (3.x)?
