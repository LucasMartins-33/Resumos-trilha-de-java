# Questões - Capítulo 04: Microsserviço de Proposta

### Questão 1: Por que a arquitetura MVC foi aplicada na estruturação dos pacotes desse microserviço?
> [!success]- Resposta
> Para separar responsabilidades. O `Controller` lida estritamente com requisições HTTP, o `Service` centraliza as regras de negócio e validações, e o `Repository` isola o acesso ao banco de dados, impedindo que requisições acessem o banco diretamente.

### Questão 2: Qual é o propósito do DTO (Data Transfer Object) na arquitetura do serviço de Proposta?
> [!success]- Resposta
> Isolar a Entidade de banco de dados (`ProposalEntity`) do formato consumido pela API e pelo Kafka. Isso previne o vazamento de informações sensíveis ou de infraestrutura (como senhas ou anotações JPA) para fora da aplicação.

### Questão 3: Por que o serviço envia para o Kafka o `ProposalDTO` (resumido) em vez do `ProposalDetailsDTO` (completo)?
> [!success]- Resposta
> Para evitar trafegar dados desnecessários pelo broker de mensagens. O microsserviço de Relatórios (Consumer) só precisa do ID da proposta, do nome do cliente e do preço ofertado; os demais campos detalhados não têm utilidade para ele.

### Questão 4: O que acontece se a implementação `ProposalServiceImpl` não for anotada com `@ApplicationScoped`?
> [!success]- Resposta
> O Quarkus não registrará a classe no contexto de injeção de dependências (CDI). Ao tentar injetá-la no `ProposalController` com `@Inject`, ocorrerá um erro de *Unsatisfied dependency* durante a inicialização.

### Questão 5: O método de remover uma proposta deve receber qual anotação obrigatória para manipular o Hibernate?
> [!success]- Resposta
> A anotação `@Transactional` (do pacote `jakarta.transaction.Transactional`), pois excluir um registro do banco altera o estado da transação.

### Questão 6: No Quarkus, qual verbo e anotação JAX-RS/Jakarta são utilizados para definir o endpoint que aceita o envio (criação) de uma nova proposta?
> [!success]- Resposta
> A anotação `@POST`.

### Questão 7: Para conectar a um banco de dados independente apenas para Propostas, qual configuração no `application.properties` define o URL de conexão JDBC?
> [!success]- Resposta
> A propriedade `quarkus.datasource.jdbc.url=jdbc:postgresql://[host]:[porta]/[nome_do_banco]`.

### Questão 8: Qual é a função da interface `Emitter<T>` do SmallRye Reactive Messaging neste projeto?
> [!success]- Resposta
> Ela permite disparar (emitir) mensagens ou eventos para um canal pré-configurado. No caso, ela é usada para enviar o objeto Java (`ProposalDTO`) para o tópico do Kafka configurado no canal.

### Questão 9: Como o Hibernate gerencia as tabelas no banco de dados automaticamente quando a propriedade `database.generation=update` é usada?
> [!success]- Resposta
> O Hibernate compara o modelo de entidades no código Java com o esquema real do banco de dados ao inicializar. Se uma tabela ou coluna não existir, ele cria ou altera a estrutura para refletir as anotações mapeadas na classe Java.

### Questão 10: Na criação do repositório, como podemos buscar uma proposta com base no nome do "Customer" através do Panache?
> [!success]- Resposta
> Implementando um método customizado no Repositório que utilize o método base `find()`. Exemplo: `find("customer", nomeDoCliente).firstResultOptional();`.
