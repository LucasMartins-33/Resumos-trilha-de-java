# Capítulo 07: Generating Source with Maven

Neste capítulo é abordada a **Geração de Código-Fonte (*Code Generation*)** automatizada durante o ciclo de build do Maven. Em vez de escrever manualmente classes repetitivas (*boilerplate*), POJOs de transporte ou conversores de dados, delegamos ao Maven e aos compiladores/processadores de anotações a tarefa de gerar essas classes antes da compilação principal (`compile`). São exploradas 4 abordagens práticas:
1. Geração a partir de esquemas XML (**JAXB / XSD**).
2. Geração a partir de esquemas JSON (**JSON Schema to POJO**).
3. Geração e enriquecimento em tempo de compilação via AST (**Project Lombok**).
4. Mapeamento declarativo de objetos de alto desempenho (**MapStruct**).

---

## 1. Geração de Classes Java a partir de XML Schema (JAXB / XSD)

O JAXB (*Java Architecture for XML Binding*) converte esquemas XML (`.xsd`) em classes Java fortemente tipadas usando o compilador `xjc`.

### 1.1. O Esquema XML: `src/main/resources/jaxb.xsd`
```xml
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           targetNamespace="guru.springframework.domain"
           xmlns="guru.springframework.domain"
           elementFormDefault="qualified">

    <xs:element name="Product">
        <xs:complexType>
            <xs:sequence>
                <xs:element name="productId" type="xs:long"/>
                <xs:element name="productDescription" type="xs:string"/>
                <xs:element name="productPrice" type="xs:decimal"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>

    <xs:element name="CreateProductRequest">
        <xs:complexType>
            <xs:sequence>
                <xs:element ref="Product"/>
                <xs:element name="apiKey" type="xs:string"/>
            </xs:sequence>
        </xs:complexType>
    </xs:element>
</xs:schema>
```

### 1.2. Configuração do Plugin no `pom.xml`
```xml
<build>
    <plugins>
        <!-- Plugin JAXB para executar o compilador xjc -->
        <plugin>
            <groupId>org.jvnet.jaxb2.maven2</groupId>
            <artifactId>maven-jaxb2-plugin</artifactId>
            <version>0.14.0</version>
            <executions>
                <execution>
                    <goals>
                        <goal>generate</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

* **Resultado da Execução (`mvn package`):**
  * O plugin lê o arquivo `.xsd` e cria as classes Java anotadas com JAXB (`@XmlRootElement`, `@XmlElement`) no diretório:
    `target/generated-sources/xjc/guru/springframework/domain/`
  * O Maven adiciona essa pasta automaticamente como raiz de código-fonte e a compila para `target/classes/`.

---

## 2. Geração de Classes Java a partir de JSON Schema

Para contratos baseados em JSON, utiliza-se o plugin `jsonschema2pojo-maven-plugin` para gerar POJOs com suporte a Jackson e Apache Commons Lang.

### 2.1. O Arquivo JSON Schema: `src/main/resources/schema/ShippingAddress.json`
```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "title": "ShippingAddress",
  "type": "object",
  "properties": {
    "street": { "type": "string" },
    "city": { "type": "string" },
    "state": { "type": "string" },
    "postalCode": { "type": "string" }
  },
  "required": ["street", "city", "postalCode"]
}
```

### 2.2. Dependências e Configuração no `pom.xml`
```xml
<dependencies>
    <!-- Utilitários para equals, hashCode e toString gerados -->
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.12.0</version>
    </dependency>
    <!-- Jackson para anotações de serialização/deserialização -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.13.0</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.jsonschema2pojo</groupId>
            <artifactId>jsonschema2pojo-maven-plugin</artifactId>
            <version>1.1.2</version>
            <configuration>
                <sourceDirectory>${basedir}/src/main/resources/schema</sourceDirectory>
                <targetPackage>guru.springframework.model</targetPackage>
                <useCommonsLang3>true</useCommonsLang3>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>generate</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

* **Propriedade Maven `${basedir}`:** Aponta para o diretório raiz absoluto do projeto em tempo de execução.
* **Nome das Classes:** O plugin utiliza o atributo `title` do JSON ou o próprio nome do arquivo `.json` como nome da classe Java gerada.

---

## 3. Project Lombok: Geração de Código em Tempo de Compilação

O **Project Lombok** intercepta a Árvore de Sintaxe Abstrata (AST) durante a compilação do Java (via processador de anotações) e injeta bytecodes diretamente na classe compilada, eliminando métodos repetitivos.

### 3.1. Dependência no `pom.xml`
```xml
<dependencies>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.30</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```
* **Por que `<scope>provided</scope>`?** O Lombok só atua durante a compilação. Em tempo de execução, os bytecodes de getters/setters já estão gravados no `.class`, dispensando o JAR do Lombok no pacote final.

### 3.2. Modelo de Classe Enxuto
```java
package guru.springframework.model;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class User {
    private String firstName;
    private String lastName;
    private String email;
}
```

* `@Data`: Agrupa `@Getter`, `@Setter`, `@ToString`, `@EqualsAndHashCode` e `@RequiredArgsConstructor`.
* `@Builder`: Gera automaticamente uma classe interna com o padrão de projeto *Builder* (`User.builder().firstName("John").build()`).
* **Configuração Obrigatória na IDE (IntelliJ):** Deve-se habilitar o processamento de anotações em:
  * *Settings -> Build, Execution, Deployment -> Compiler -> Annotation Processors -> Enable annotation processing*.

---

## 4. MapStruct: Mapeamento de Objetos de Alta Performance

O **MapStruct** gera código Java nativo em tempo de compilação para converter dados entre objetos (ex: entidades JPA de banco de dados $\leftrightarrow$ DTOs / Command Objects de API/Web). É imensamente mais rápido e seguro que conversores baseados em reflexão (como o antigo ModelMapper).

### 4.1. Configuração do `pom.xml`
O processador de anotações do MapStruct deve ser registrado na configuração do `maven-compiler-plugin`:

```xml
<properties>
    <org.mapstruct.version>1.5.5.Final</org.mapstruct.version>
</properties>

<dependencies>
    <!-- Anotações do MapStruct (ex: @Mapper) -->
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>${org.mapstruct.version}</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.11.0</version>
            <configuration>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>${org.mapstruct.version}</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

### 4.2. Declarando a Interface do Mapper
```java
package guru.springframework.mappers;

import guru.springframework.domain.User;
import guru.springframework.model.UserCommand;
import org.mapstruct.Mapper;
import org.mapstruct.factory.Mappers;

@Mapper
public interface UserMapper {
    UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);

    UserCommand userToUserCommand(User user);
    User userCommandToUser(UserCommand userCommand);
}
```

* **O que o Maven faz:** Durante o `compile`, o processador gera o arquivo `UserMapperImpl.java` em `target/generated-sources/annotations/` contendo código Java puro com verificações de `null` e chamadas diretas de getters e setters.

---

## 5. Apêndice — Atualizações & Boas Práticas Modernas

### 1. Migração do JAXB para o Namespace Jakarta EE (Java 17 / 21)
Na aula, o instrutor utiliza `javax.xml.bind` e o plugin `org.jvnet.jaxb2.maven2:maven-jaxb2-plugin` (versão `0.14.0`), pois o Java 11 removeu os módulos corporativos do JDK padrão.
* **Hoje:** Com a transição do Java EE para o **Jakarta EE**, o pacote `javax.*` virou `jakarta.*`.
* Para projetos modernos com Java 17 ou 21 e Spring Boot 3+, deve-se usar o plugin oficial do Jakarta:
  ```xml
  <plugin>
      <groupId>org.codehaus.mojo</groupId>
      <artifactId>jaxb2-maven-plugin</artifactId>
      <version>3.1.0</version>
      <executions>
          <execution>
              <goals>
                  <goal>xjc</goal>
              </goals>
          </execution>
      </executions>
  </plugin>
  ```

### 2. Armadilha Crítica: Usando Lombok + MapStruct Juntos
Quando configuramos `<annotationProcessorPaths>` explicitamente no `maven-compiler-plugin` (como exigido pelo MapStruct), o compilador do Maven **para de procurar processadores no classpath de dependências normais**. 
* Consequência: O Lombok para de funcionar e o MapStruct não encontra os getters/setters que o Lombok geraria.
* **A Solução Moderna Oficial:** Registrar ambos no `<annotationProcessorPaths>` juntamente com o conector `lombok-mapstruct-binding`:
  ```xml
  <configuration>
      <annotationProcessorPaths>
          <!-- 1. Lombok Processor -->
          <path>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
              <version>1.18.30</version>
          </path>
          <!-- 2. Binding que sincroniza a ordem de execução entre Lombok e MapStruct -->
          <path>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok-mapstruct-binding</artifactId>
              <version>0.2.0</version>
          </path>
          <!-- 3. MapStruct Processor -->
          <path>
              <groupId>org.mapstruct</groupId>
              <artifactId>mapstruct-processor</artifactId>
              <version>1.5.5.Final</version>
          </path>
      </annotationProcessorPaths>
  </configuration>
  ```

### 3. Java Records (Java 16+) vs Lombok
Para DTOs e objetos de transporte de dados somente-leitura (como o `UserCommand` ou respostas de APIs), o Java nativo introduziu os **Records**:
```java
public record UserRecord(String firstName, String lastName, String email) {}
```
* Records dispensam dependências externas, bibliotecas como Lombok e geram construtor canônico, getters (no formato `firstName()`), `equals()`, `hashCode()` e `toString()` nativamente pela linguagem.
* O MapStruct possui suporte nativo para mapear entre Entidades e Java Records a partir da versão 1.4+.

### 4. Geração Moderna de Contratos de API: OpenAPI Generator
Embora esquemas XSD e JSON Schema ainda existam em sistemas de mensageria e integrações corporativas, o padrão predominante na indústria moderna para geração de código a partir de contratos de API REST é a especificação **OpenAPI 3.0** via o plugin **`openapi-generator-maven-plugin`**, que gera tanto os modelos de dados quanto as interfaces dos controladores Spring MVC / JAX-RS diretamente a partir de um arquivo `openapi.yaml`.
