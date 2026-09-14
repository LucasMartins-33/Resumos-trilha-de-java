# Questões Teóricas - Capítulo 07 (Generating Source with Maven)

**1. Geração de Código na Fase `generate-sources`:** Por que ferramentas de integração de dados e arquitetura orientada a serviços costumam gerar código Java a partir de contratos neutros (como XSD ou JSON Schema) durante a fase `generate-sources` do Maven, em vez de manter essas classes escritas manualmente no Git?
<details>
<summary>👀 Ver Resposta</summary>

Porque o contrato de dados (XSD, JSON Schema, OpenAPI ou Protobuf) é a "fonte única da verdade" (*Single Source of Truth*). Ao automatizar a geração de classes POJO no ciclo de build do Maven, garante-se que os modelos Java estejam sempre perfeitamente sincronizados com o esquema canônico da empresa. Se o esquema for alterado, o código gerado reflete a mudança no próximo build, eliminando retrabalho manual propenso a erros de digitação e divergências de tipagem.
</details>

**2. Destino Padrão de Código Gerado (`target/generated-sources`):** Por que os plugins geradores de código do Maven colocam as classes criadas sob a pasta `target/generated-sources/` e nunca diretamente dentro de `src/main/java`?
<details>
<summary>👀 Ver Resposta</summary>

Porque classes geradas são artefatos voláteis que **não devem ser commitadas no controle de versão (Git)**. Manter o código gerado dentro da pasta `target/generated-sources/` garante que, ao executar `mvn clean`, essas classes sejam limpas e recriadas de forma fresca. Além disso, o plugin adiciona esse caminho dinamicamente à lista de diretórios-fonte do compilador, sem poluir o diretório original `src/main/java` mantido pelos programadores.
</details>

**3. O Plugin `jaxb2-maven-plugin`:** O que é o utilitário **XJC** gerenciado pelo plugin JAXB do Maven e qual é a sua função ao processar arquivos `.xsd` (XML Schema Definition)?
<details>
<summary>👀 Ver Resposta</summary>

O XJC (*XML-to-Java Compiler*) é a ferramenta interna de ligação do JAXB. Ao receber um esquema XML (`.xsd`), o XJC analisa as definições de tipos complexos, restrições e elementos, gerando automaticamente classes Java anotadas com anotações padrão do JAXB (como `@XmlRootElement`, `@XmlElement`, `@XmlType`), permitindo a serialização e desserialização instantânea (*marshalling* e *unmarshalling*) entre objetos Java e documentos XML.
</details>

**4. O Plugin `jsonschema2pojo-maven-plugin`:** Como o plugin `jsonschema2pojo` lida com definições de esquema JSON e quais anotações de bibliotecas populares de serialização (como Jackson ou Gson) ele pode adicionar automaticamente aos POJOs gerados?
<details>
<summary>👀 Ver Resposta</summary>

O plugin lê arquivos JSON Schema (`.json`) ou até mesmo documentos JSON de exemplo e gera classes POJO ricas em Java. Ele pode ser configurado via tag `<annotationStyle>` para injetar anotações de bibliotecas de mercado como **Jackson 2** (ex: `@JsonProperty`, `@JsonInclude`), **Gson** ou validações da Jakarta Bean Validation (ex: `@NotNull`, `@Size`), permitindo integração imediata com APIs REST.
</details>

**5. Processamento de Anotações em Tempo de Compilação (APT):** Como ferramentas como o **Project Lombok** e o **MapStruct** realizam suas transformações no código Java? Eles operam através de bytecode gerado em runtime ou em tempo de compilação?
<details>
<summary>👀 Ver Resposta</summary>

Ambos operam estritamente em **tempo de compilação** através do mecanismo de *Annotation Processing* (APT) padronizado da JVM (JSR 269). Durante a fase `compile`, o compilador `javac` intercepta as anotações do Lombok (`@Getter`, `@Setter`, `@Builder`) e modifica a Árvore Sintática Abstrata (AST) em memória, injetando os métodos diretamente no bytecode `.class`. O MapStruct, por sua vez, lê interfaces anotadas com `@Mapper` e gera o código-fonte Java puro de implementação dos mapeamentos antes da geração final do bytecode, garantindo custo zero de performance em tempo de execução (*zero runtime overhead*).
</details>

**6. A Armadilha de Compilação Lombok + MapStruct:** Por que o uso conjunto do Project Lombok e do MapStruct frequentemente falha na compilação padrão do Maven se não houver uma configuração explícita no `maven-compiler-plugin`?
<details>
<summary>👀 Ver Resposta</summary>

O MapStruct precisa ler os métodos Getters, Setters e Construtores das classes de modelo para gerar a lógica de mapeamento. Porém, se o Lombok não tiver sido executado **antes** do MapStruct, esses métodos ainda não existem na classe, fazendo o MapStruct emitir erros de que as propriedades não foram encontradas. A solução é declarar explicitamente a ordem de execução no `<annotationProcessorPaths>` do `maven-compiler-plugin` e incluir o artefato facilitador `lombok-mapstruct-binding`.
</details>

**7. A Biblioteca Facilitadora `lombok-mapstruct-binding`:** Qual é a função do artefato `org.projectlombok:lombok-mapstruct-binding` dentro do `<annotationProcessorPaths>`?
<details>
<summary>👀 Ver Resposta</summary>

Essa biblioteca atua como uma ponte de sincronização entre os processadores de anotação do Lombok e do MapStruct. Ela garante formalmente ao compilador `javac` que o Lombok processe e finalize a injeção de todos os acessores (Getters/Setters) nas classes antes que o MapStruct tente analisar os tipos e gerar o código das classes de conversão, eliminando condições de corrida e erros de resolução durante o build.
</details>

**8. O Padrão `componentModel="spring"` no MapStruct:** O que a configuração `<compilerArg>-Amapstruct.defaultComponentModel=spring</compilerArg>` faz nas classes geradas pelo MapStruct?
<details>
<summary>👀 Ver Resposta</summary>

Essa configuração instrui o processador do MapStruct a anotar todas as classes de implementação geradas com a anotação `@Component` do Spring Framework. Dessa forma, as classes mapeadoras tornam-se Spring Beans gerenciados pelo contêiner IoC, podendo ser facilmente injetadas em Services ou Controllers através de `@Autowired` ou injeção de dependência por construtor.
</details>

**9. A Grande Migração Jakarta EE (`javax.*` para `jakarta.*`):** Por que plugins legados de JAXB que geravam código sob o pacote `javax.xml.bind.*` quebram em aplicações modernas com Java 17+ e Spring Boot 3+? Qual é a substituição correta?
<details>
<summary>👀 Ver Resposta</summary>

Devido à transferência da governança do Java EE da Oracle para a Eclipse Foundation, a marca registrada "Java" não pôde ser utilizada, forçando a renomeação universal dos pacotes corporativos de `javax.*` para `jakarta.*` no Jakarta EE 9/10. Em Java 17+ e Spring Boot 3+, o suporte antigo a `javax.xml.bind` foi removido, sendo obrigatório atualizar o plugin para versões modernas compatíveis com o padrão Jakarta (como o artefato `com.sun.xml.bind:jaxb-impl` v4.x e coordenadas Jakarta).
</details>

**10. Java Records vs Geradores de Código:** Com a introdução dos **Java Records** (recurso padrão a partir do Java 16), qual necessidade de geração de código ou uso de anotações (como `@Data` do Lombok) é drasticamente reduzida em novos projetos?
<details>
<summary>👀 Ver Resposta</summary>

Os Records eliminam nativamente a necessidade de escrever ou gerar código para classes imutáveis de transferência de dados (DTOs). Ao declarar `public record UsuarioDTO(String nome, String email) {}`, o próprio compilador Java sintetiza automaticamente atributos privados e finais, construtor canônico, métodos acessores (Getters), `equals()`, `hashCode()` e `toString()`, tornando redundantes bibliotecas geradoras para esse propósito específico.
</details>
