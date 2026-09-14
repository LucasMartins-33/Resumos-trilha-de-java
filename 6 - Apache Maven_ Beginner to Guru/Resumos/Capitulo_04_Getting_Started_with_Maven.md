# Capítulo 04: Getting Started with Maven

Neste capítulo, é construído o primeiro projeto prático utilizando o Apache Maven, saindo dos comandos manuais de compilação e empacotamento vistos no capítulo anterior para a automação estruturada. É apresentado o arquivo central `pom.xml`, a estrutura padrão de diretórios, a inclusão de dependências externas e a integração com o IntelliJ IDEA.

---

## 1. O Arquivo POM (`pom.xml`)

O arquivo **POM** (*Project Object Model*) é o coração de qualquer projeto Maven. Escrito em XML, ele descreve a identidade do projeto, suas configurações, dependências e plugins de build.

### 1.1. Estrutura Mínima de um `pom.xml`
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Coordenadas Maven do Projeto -->
    <groupId>guru.springframework</groupId>
    <artifactId>hello-world</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- Propriedades e Configurações de Compilação -->
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>11</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
    </properties>
</project>
```

### 1.2. Dissecando as Tags Fundamentais

* `<modelVersion>`: Especifica a versão do modelo do descritor POM. Para o Maven 2 e Maven 3, o valor padrão obrigatório é sempre `4.0.0`.
* **Coordenadas Maven (GAV):**
  * `<groupId>`: Identificador único da organização, empresa ou pacote raiz do projeto. Por convenção mundial, adota o nome de domínio reverso (ex: `guru.springframework`, `com.minhaempresa`).
  * `<artifactId>`: Nome do artefato / projeto específico (geralmente em minúsculas separado por hifens, ex: `hello-world`). Esse será o nome base do arquivo `.jar` gerado.
  * `<version>`: Versão atual do artefato. O sufixo `-SNAPSHOT` indica uma versão **em desenvolvimento** (mutável), enquanto versões sem `-SNAPSHOT` (ex: `1.0.0`) são versões finais estáveis (*releases* imutáveis).
* `<properties>`: Seção para declarar variáveis reutilizáveis no POM:
  * `project.build.sourceEncoding`: Codificação usada ao ler arquivos-fonte (evita que acentuações quebrem compilações entre Windows/Linux). O padrão é `UTF-8`.
  * `maven.compiler.source` e `maven.compiler.target`: Informam ao compilador do Maven qual versão do Java aceitar no código e qual versão de bytecode gerar.

---

## 2. Estrutura Padrão de Diretórios do Maven (*Standard Directory Layout*)

O Maven baseia-se no princípio de **Convenção sobre Configuração** (*Convention over Configuration*). Ele espera encontrar os arquivos em locais padronizados sem que você precise configurá-los:

```
hello-world/
├── pom.xml                     # Arquivo de configuração raiz
├── src/
│   ├── main/
│   │   ├── java/               # Código-fonte Java de produção (.java)
│   │   └── resources/          # Arquivos de configuração, properties, XMLs de produção
│   └── test/
│       ├── java/               # Código-fonte Java dos testes unitários/integração
│       └── resources/          # Arquivos e configurações específicos para os testes
└── target/                     # Diretório de saída gerado pelo Maven (NUNCA commitado no Git)
    ├── classes/                # Bytecodes compilados (.class)
    └── hello-world-1.0-SNAPSHOT.jar # Artefato final empacotado
```

> **Atenção:** Se seus arquivos `.java` estiverem soltos na raiz ou fora de `src/main/java`, o compilador do Maven simplesmente não os encontrará durante o build.

---

## 3. Comandos e Ciclo de Vida do Maven

Os comandos do Maven acionam **fases** e **metas** (*goals*) do ciclo de vida:

| Comando | O que faz |
| :--- | :--- |
| `mvn clean` | Executa o plugin `maven-clean-plugin`. Deleta o diretório `target/` e todos os arquivos gerados em builds anteriores. |
| `mvn compile` | Compila o código-fonte de produção de `src/main/java` e salva os `.class` em `target/classes`. |
| `mvn package` | Executa todas as fases anteriores (validação, compilação, testes) e empacota as classes compiladas no formato especificado (padrão: `.jar` em `target/`). |
| `mvn clean package` | Combina a limpeza com um novo empacotamento completo do zero (garante que artefatos antigos não interfiram). |

### 3.1. Anatomia do JAR Gerado
Ao inspecionar o arquivo `target/hello-world-1.0-SNAPSHOT.jar` via `unzip -l`, observa-se:
1. As classes compiladas (`HelloWorld.class`).
2. `META-INF/MANIFEST.MF`: Metadados indicando o autor do build, versão do JDK e versão do Maven.
3. `META-INF/maven/<groupId>/<artifactId>/pom.xml` e `pom.properties`: O Maven inclui automaticamente uma cópia do próprio POM e as propriedades exatas do build dentro do JAR, garantindo rastreabilidade do artefato.

---

## 4. Gerenciamento de Dependências Externas

Para utilizar bibliotecas de terceiros (como o Apache Commons Lang para manipular strings), adiciona-se o bloco `<dependencies>` no `pom.xml`.

### 4.1. Declarando a Dependência
```xml
<dependencies>
    <dependency>
        <groupId>org.apache.commons</groupId>
        <artifactId>commons-lang3</artifactId>
        <version>3.8.1</version>
    </dependency>
</dependencies>
```

### 4.2. O que o Maven faz automaticamente:
1. **Download:** O Maven consulta o Repositório Central (*Maven Central*) na internet, baixa o JAR `commons-lang3-3.8.1.jar` e o armazena no repositório local da máquina (geralmente em `~/.m2/repository`).
2. **Classpath de Compilação:** Durante a fase de compilação, o Maven injeta automaticamente a dependência no parâmetro `-classpath` do `javac`.
3. **Thin JAR por Padrão:** O JAR padrão gerado por `mvn package` é um **Thin JAR** (contém apenas o código do seu projeto; a biblioteca externa não é descompactada para dentro dele).

### 4.3. Uso no Código Java
```java
package guru.springframework;

import org.apache.commons.lang3.StringUtils;

public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World");
        // StringUtils.capitalize capitaliza apenas a primeira letra da frase
        System.out.println(StringUtils.capitalize("hello world")); 
        // Saída: "Hello world"
    }
}
```

---

## 5. Integração com o IntelliJ IDEA

Ao trabalhar com IDEs profissionais como o IntelliJ:
1. **Criação de Projetos:** Pode-se iniciar um projeto escolhendo *New Project -> Maven* (sem arquétipo para projetos básicos puros).
2. **Janela de Ferramentas Maven (*Maven Tool Window*):** Permite executar fases do ciclo de vida (`clean`, `compile`, `package`) com duplo-clique, sem sair da IDE.
3. **Arquivo `.iml`:** Arquivo gerado pelo IntelliJ para controle interno de módulos da IDE. **Não deve ser versionado no Git** (deve constar no `.gitignore`).
4. **Resolução de Inconsistências:** Se a IDE apresentar erros de código vermelho enquanto o terminal compila com sucesso, execute um `mvn clean` seguido de um reload no Maven para forçar a IDE a reconstruir seu índice.

---

## 6. Apêndice — Atualizações & Boas Práticas Modernas

### 1. Substituição de `source`/`target` por `<maven.compiler.release>` (Java 9+)
Nas versões recentes do Java (Java 9+ até Java 17, 21 e superiores), a forma recomendada de configurar o compilador não é mais usando `maven.compiler.source` e `target`, mas sim:

```xml
<properties>
    <maven.compiler.release>17</maven.compiler.release>
</properties>
```
* **Por que mudou?** A diretiva `--release` do `javac` define a versão da linguagem, a versão do bytecode e, crucialmente, restringe as APIs disponíveis à versão escolhida da plataforma, impedindo que métodos que só existem em JDKs mais novos sejam acidentalmente chamados.

### 2. Sincronização Maven no IntelliJ IDEA Moderno
* Em versões antigas do IntelliJ, havia a opção explícita de *"Import Maven projects automatically"*.
* Nas versões modernas (2020+), o IntelliJ removeu essa caixa de diálogo antiga. Agora ele monitora o `pom.xml` automaticamente e exibe um pequeno botão flutuante com o ícone do Maven (*Reload All Maven Projects* — atalho `Ctrl + Shift + O` no Linux/Windows ou `Cmd + Shift + I` no macOS) ou realiza o sincronismo silenciosamente em segundo plano ao salvar o arquivo.

### 3. Diretórios Vazios no Git: Convenção do `.gitkeep`
* No vídeo, o instrutor adicionou arquivos dummy como `package-info.java` e `application.properties` para forçar o Git a rastrear pastas vazias.
* A convenção universal da comunidade de desenvolvimento é criar um arquivo vazio oculto chamado `.gitkeep` (ex: `src/main/resources/.gitkeep`).

### 4. Adoção Padrão do Maven Wrapper (`mvnw`)
* Em vez de exigir que todo desenvolvedor ou servidor de CI instale o Maven manualmente no sistema operacional com versões potencialmente divergentes, projetos modernos utilizam o **Maven Wrapper**:
  ```bash
  # No Linux/macOS:
  ./mvnw clean package

  # No Windows:
  mvnw.cmd clean package
  ```
  O wrapper garante que todos na equipe utilizem exatamente a mesma versão do Maven.
