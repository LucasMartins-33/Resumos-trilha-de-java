# Capítulo 01: Introdução ao Quarkus, Microserviços e Kafka

Este documento contém o resumo estruturado e atualizado do **Capítulo 01**, abordando os conceitos fundamentais sobre APIs REST, Microserviços, Apache Kafka e a evolução do ecossistema Java com o Quarkus.

---

## 1. Visão Geral da Arquitetura
O curso tem como objetivo construir uma aplicação backend corporativa baseada em **Microserviços**, utilizando:
- **Quarkus** como framework Java principal.
- **Apache Kafka** para mensageria e comunicação assíncrona orientada a eventos.
- **Keycloak** para autenticação e autorização, utilizando tokens **JWT**.
- **APIs REST** para comunicação síncrona.

---

## 2. Apache Kafka: Mensageria e Eventos
O Apache Kafka é uma plataforma de streaming de eventos distribuída, utilizada por grandes empresas (Netflix, Uber, LinkedIn) para lidar com um fluxo massivo de dados (Big Data, Machine Learning, Analytics, etc.). 

### Como o Kafka resolve o problema de comunicação?
Em uma arquitetura de microserviços, a comunicação direta via HTTP (APIs) entre muitos serviços pode gerar gargalos e dependência excessiva (acoplamento). O Kafka atua como um intermediário (Broker):
- **Produtores (Producers):** Microserviços que geram uma informação (ex: "Pagamento Aprovado") e enviam essa mensagem para o Kafka. Após o envio, o produtor está livre, não precisando esperar resposta dos outros serviços.
- **Tópicos (Topics):** Canais onde as mensagens são armazenadas dentro do Kafka. A mensagem fica retida por um período configurável (o padrão são 7 dias).
- **Partições (Partitions):** Subdivisões dos tópicos que permitem balancear a carga e aumentar a escalabilidade.
- **Consumidores (Consumers):** Aplicações "assinam" os tópicos para receber e processar as mensagens em tempo real (ex: Serviço de Logística, Serviço Antifraude, Analytics).
- **Zookeeper / KRaft:** O professor cita o Zookeeper como orquestrador do cluster do Kafka. 
  * ⚠️ **Atualização Importante:** Nas versões mais recentes do Kafka, o Zookeeper está sendo substituído pelo **KRaft** (Kafka Raft), que simplifica a arquitetura gerenciando os metadados internamente sem a necessidade de um serviço externo.

---

## 3. APIs REST e Comunicação HTTP
As APIs (Application Programming Interface) surgiram para permitir a integração e o compartilhamento seguro de informações entre diferentes sistemas e empresas. 

A comunicação REST utiliza os verbos do protocolo HTTP:
- `GET`: Buscar/recuperar uma informação.
- `POST`: Enviar/criar um novo recurso (ex: login, cadastro).
- `DELETE`: Remover um recurso.
- `PUT` / `PATCH`: Atualizar um recurso existente.
  * ⚠️ **Correção da Aula:** A transcrição mencionou o método *"PUSH"*, mas o nome correto no protocolo HTTP para atualização de recursos é **PUT** (atualização completa) ou **PATCH** (atualização parcial).

---

## 4. Monolitos vs Microserviços
- **Monolito:** Uma única aplicação contendo todos os módulos de negócio (pagamento, usuário, produto) compartilhando o mesmo banco de dados. 
  - *Problema:* Um erro grave em um módulo ou no banco derruba o sistema inteiro (**Ponto Único de Falha - SPOF**). Escalabilidade é limitada e implantações (deploys) são pesadas.
- **Microserviços:** Cada domínio do negócio é uma aplicação separada com seu próprio banco de dados.
  - *Vantagem:* Resiliência (se o serviço de pagamento cair, o de produtos continua funcionando), escalabilidade independente, e entregas mais rápidas.
  
---

## 5. Quarkus: O Ecossistema Java Subatômico e Supersônico
O Java tradicional (com sua JVM padrão) carrega dezenas de dependências em tempo de execução, consumindo muita memória RAM e demorando segundos (ou minutos) para inicializar. No mundo de *Cloud* e *Containers Docker*, onde microserviços precisam subir instantaneamente e consumir pouca RAM, o Java estava perdendo espaço para outras linguagens.

O **Quarkus** resolve isso transferindo muito do trabalho para o **tempo de compilação (Build Time)**. 
Ele suporta a criação de executáveis nativos via **GraalVM** (compilação AOT - Ahead-Of-Time).

### Vantagens do Quarkus:
- Inicialização na casa de milissegundos (ex: `0.016s`).
- Consumo baixíssimo de memória (nível "subatômico").
- Integração nativa com Kubernetes e Docker.
- Suporta bibliotecas tradicionais (Hibernate, RESTEasy, etc.).

### Sintaxe e Comandos Novos (Quarkus + GraalVM)
*(Nota: Durante a aula, ocorreu um erro de transcrição onde o comando do Maven foi escrito como "O mês em que de menos penates" e o nome do executável como "Cold Wife Corpus". Aqui está a sintaxe real)*

**1. Para gerar o build de um binário nativo (usando GraalVM):**
```bash
mvn package -Pnative
```
*Dica: Caso não tenha o GraalVM instalado localmente, você pode delegar o build nativo para o Docker passando a flag `-Dquarkus.native.container-build=true`.*

**2. Para executar o binário nativo gerado:**
Após o comando acima, um executável nativo é criado na pasta `target`. Para executá-lo, basta rodar diretamente no terminal (sem precisar do comando `java -jar`):
```bash
./target/nome-do-projeto-1.0.0-SNAPSHOT-runner
```

---

## 6. Ambiente de Desenvolvimento (Softwares Necessários)
O professor listou as ferramentas base. 

⚠️ **Atualizações Importantes para as Versões Atuais:**
1. **Java (JDK):** O professor recomendou Java 11. No entanto, o **Quarkus nas versões atuais (3.x) exige no mínimo o Java 17**. Recomenda-se instalar o Java 17 ou 21.
2. **Apache Kafka:** Recomenda-se rodar via Docker para facilitar o setup local.
3. **Maven:** Gerenciador de pacotes e dependências.
4. **Docker / Docker Desktop:** Essencial para rodar o Kafka e o banco de dados sem poluir o sistema operacional host.
5. **IDE:** IntelliJ IDEA ou Eclipse.
6. **PostgreSQL:** Banco de dados relacional, também recomendável rodar via Docker (`docker run --name postgres ...`).
