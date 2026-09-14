# Questões Práticas - Capítulo 07 (Generating Source with Maven)

🟢 Nível 1: Criando um Esquema XML (XSD) Mínimo
Cenário: Você precisa estabelecer um contrato formal de integração via XML para cadastro de produtos.
Sua Tarefa:
* Crie a pasta `src/main/resources/xsd`.
* Dentro dela, crie o arquivo `produto.xsd` definindo um elemento raiz `<produto>` contendo os campos `<id>` (inteiro), `<nome>` (string) e `<preco>` (decimal).
* Valide a sintaxe básica das tags `<xs:schema>`, `<xs:element>` e `<xs:complexType>`.

🟡 Nível 2: Configurando o Plugin JAXB para Geração de Código
Cenário: Você não quer escrever os POJOs Java na mão; o Maven deve gerá-los a partir do arquivo XSD.
Sua Tarefa:
* No `pom.xml`, configure o plugin `org.codehaus.mojo:jaxb2-maven-plugin:3.1.0`.
* Dentro de `<executions>`, vincule o goal `xjc` à fase `generate-sources`.
* Especifique o diretório do esquema apontando para `src/main/resources/xsd`.
* Execute no terminal: `mvn generate-sources`.
* Verifique se as classes Java anotadas com JAXB foram geradas sob `target/generated-sources/jaxb/`.

🟠 Nível 3: Consumindo o Código Gerado pelo JAXB
Cenário: Com as classes geradas na pasta `target/generated-sources`, você precisa utilizá-las no código de produção.
Sua Tarefa:
* Na classe `src/main/java/com/minhaempresa/App.java`, importe a classe `Produto` gerada pelo JAXB.
* Instancie um `Produto`, atribua valores com os setters gerados e imprima o nome do produto no console.
* Execute `mvn clean compile` e comprove que o compilador encontrou as classes geradas automaticamente sem erros.

🔴 Nível 4: Gerando POJOs a partir de JSON Schema
Cenário: Em um projeto de microsserviços REST, o contrato foi definido no formato JSON Schema (`usuario-schema.json`).
Sua Tarefa:
* Crie o arquivo `src/main/resources/schema/usuario.json` descrevendo as propriedades `id`, `nome` e `email`.
* Adicione o plugin `org.jsonschema2pojo:jsonschema2pojo-maven-plugin:1.2.1`.
* Configure o pacote de destino como `com.minhaempresa.dto` e o estilo de anotação como `jackson2`.
* Execute `mvn generate-sources` e inspecione a classe gerada `Usuario.java` sob `target/generated-sources/jsonschema2pojo/`.

🟣 Nível 5: Adicionando o Project Lombok
Cenário: Sua equipe quer eliminar código repetitivo de Getters, Setters, construtores e `toString()` nas classes escritas manualmente.
Sua Tarefa:
* Adicione a dependência `org.projectlombok:lombok:1.18.30` com escopo `provided` no `pom.xml`.
* Crie uma classe `Cliente.java` em `src/main/java` anotada com `@Getter`, `@Setter` e `@ToString`.
* Escreva apenas os atributos privados `id` e `nome` (sem métodos).
* Em outra classe, instancie `Cliente`, use os métodos `setNome(...)` e `getNome()`, e execute `mvn compile` para validar o processamento em tempo de compilação.

🟤 Nível 6: Adicionando o MapStruct para Mapeamento de DTOs
Cenário: Você precisa converter objetos de entidade `Cliente` para o DTO `ClienteDTO` com alta performance em tempo de compilação.
Sua Tarefa:
* Adicione as dependências do `org.mapstruct:mapstruct:1.5.5.Final`.
* Crie a classe `ClienteDTO.java`.
* Crie a interface `ClienteMapper.java` anotada com `@Mapper`.
* Defina a assinatura do método: `ClienteDTO toDto(Cliente cliente)`.

🔵 Nível 7: O Conflito de Compilação Lombok + MapStruct
Cenário: Ao compilar o projeto contendo Lombok e MapStruct juntos, o MapStruct falha dizendo que não encontra os métodos getters/setters da classe `Cliente`.
Sua Tarefa:
* Execute `mvn clean compile` e analise a mensagem de erro do compilador.
* Entenda a causa: o compilador rodou o MapStruct antes do Lombok gerar os métodos.

🟢 Nível 8: Resolvendo o Conflito no `maven-compiler-plugin`
Cenário: Você precisa instruir o compilador Java sobre a ordem exata de execução dos processadores de anotação.
Sua Tarefa:
* No `pom.xml`, configure o `maven-compiler-plugin`.
* Dentro de `<configuration><annotationProcessorPaths>`, adicione nesta ordem:
  1. `org.mapstruct:mapstruct-processor:1.5.5.Final`
  2. `org.projectlombok:lombok:1.18.30`
  3. `org.projectlombok:lombok-mapstruct-binding:0.2.0`
* Execute `mvn clean compile` e confirme o `BUILD SUCCESS`.

🟡 Nível 9: Configurando Injeção Spring no MapStruct
Cenário: Você deseja que os mappers gerados pelo MapStruct sejam injetáveis como Spring Beans via `@Autowired`.
Sua Tarefa:
* No `maven-compiler-plugin`, adicione a tag `<compilerArgs>` com o argumento:
  `<compilerArg>-Amapstruct.defaultComponentModel=spring</compilerArg>`.
* Execute `mvn clean compile`.
* Abra o arquivo gerado em `target/generated-sources/annotations/.../ClienteMapperImpl.java` e verifique a presença da anotação `@Component` do Spring no topo da classe.

🟠 Nível 10: Substituindo DTOs com Java 16+ Records
Cenário: Você deseja modernizar a base de código do projeto substituindo classes de DTOs com Lombok por Java Records nativos.
Sua Tarefa:
* Certifique-se de que `<maven.compiler.release>` esteja em 17 ou superior.
* Crie o registro: `public record ProdutoRecord(Long id, String nome, BigDecimal preco) {}`.
* Ajuste a interface do MapStruct para mapear entre `Produto` e `ProdutoRecord`.
* Execute `mvn clean test` e comprove a interoperabilidade nativa do MapStruct com Java Records sem necessidade de anotações do Lombok.
