# Capítulo 03: Compiling Java

Neste capítulo, é apresentado o que acontece "por baixo dos panos" antes de delegarmos todo o ciclo de vida da aplicação para o Apache Maven. Compreender o processo de compilação e empacotamento manual no terminal ajuda a entender exatamente quais dores o Maven resolve: gerenciamento de classpath, resolução de conflitos de dependências ("JAR Hell"), padronização e automação de builds.

---

## 1. O Processo de Compilação no Java

O ciclo de vida do código-fonte em plataformas JVM baseia-se em duas etapas fundamentais:

1. **Compilação (`javac`):** O compilador converte os arquivos de código-fonte (`.java`) em arquivos binários intermediários chamados **bytecodes** (`.class`).
2. **Execução (`java` / JVM):** A Máquina Virtual Java (JVM) interpreta e/ou compila (via JIT - *Just-In-Time Compiler*) os bytecodes para instruções nativas do sistema operacional e hardware subjacente.

```
[Código-Fonte .java] ──( javac )──> [Bytecode .class] ──( JVM )──> [Instruções de Máquina / CPU]
 (Independente de SO)               (Independente de SO)           (Específico do SO: Linux, Win, Mac)
```

### Linguagens Alternativas na JVM
O compilador e a JVM não são exclusivos do Java. Qualquer linguagem cujas ferramentas produzam bytecodes em conformidade com as especificações da JVM pode ser executada pela JVM e gerenciada pelo Maven. As mais populares são:
* **Kotlin** (`.kt`)
* **Scala** (`.scala`)
* **Groovy** (`.groovy`)

---

## 2. Tipos de Empacotamento e Evolução da Arquitetura

Aplicações reais são compostas por centenas ou milhares de classes compiladas, arquivos de configuração e recursos estáticos. O empacotamento organiza esses elementos em um único arquivo compactado (formato ZIP padronizado).

| Formato | Nome Completo | Descrição | Cenário Típico de Uso |
| :--- | :--- | :--- | :--- |
| **JAR** | *Java ARchive* | Pacote ZIP contendo classes compiladas (`.class`), recursos e metadados no diretório `META-INF`. | Bibliotecas reutilizáveis ou aplicações desktop/CLI simples. |
| **WAR** | *Web Application aRchive* | Pacote contendo classes, bibliotecas terceiras (em `WEB-INF/lib`) e recursos web (HTML, JSP, JS, CSS). | Aplicações web tradicionais implantadas em Servidores de Aplicação / Servlet Containers (Tomcat, Jetty). |
| **EAR** | *Enterprise ARchive* | Superpacote contendo múltiplos WARs e JARs empresariais (EJBs). | Aplicações corporativas monolíticas legadas (WebLogic, WebSphere, WildFly). |
| **Fat JAR / Uber JAR** | *Executable / Fat JAR* | JAR executável que inclui o código da aplicação **mais todas as suas dependências descompactadas e um servidor web embutido** (ex: Tomcat embedded). | Padrão da indústria moderna (Spring Boot, Quarkus, Micronaut). |
| **Container Docker** | *OCI Container Image* | Imagem que empacota o sistema operacional base, a JVM e a aplicação (geralmente o Fat JAR). | Microsserviços e deploys em nuvem (Kubernetes, AWS ECS, GCP Cloud Run). |

---

## 3. Prática de Linha de Comando: Compilação e Execução Básica

### 3.1. Código-fonte: `HelloWorld.java`
```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

### 3.2. Compilando com `javac`
O comando `javac` recebe o caminho para os arquivos `.java` e gera os respectivos `.class`:
```bash
javac HelloWorld.java
```
* **Resultado:** É gerado o arquivo `HelloWorld.class` no mesmo diretório.
* **Observação:** O arquivo `.class` é binário. Caso seja aberto em um editor de texto, conterá caracteres ilegíveis e strings literais embutidas (iniciando pelos magic bytes `0xCAFEBABE`).

### 3.3. Executando com `java`
Para executar uma classe compilada pela JVM:
```bash
java HelloWorld
```
> **Atenção à Sintaxe:** Passa-se o **nome da classe totalmente qualificado**, sem a extensão `.class`. Não use `java HelloWorld.class`.

---

## 4. Criando e Executando Arquivos JAR

### 4.1. Sintaxe do comando `jar`
O utilitário `jar` (fornecido junto com o JDK) cria e manipula arquivos compactados no formato Java Archive:
```bash
jar cf myjar.jar HelloWorld.class
```

**Dissecando as flags:**
* `c` (*create*): Cria um novo arquivo de arquivo compactado.
* `f` (*file*): Especifica o nome do arquivo JAR a ser gerado (neste caso, `myjar.jar`).
* `HelloWorld.class`: Arquivo(s) que serão incluídos dentro do arquivo compactado.

### 4.2. Executando uma classe dentro do JAR
Como o JAR acima não definiu qual classe possui o método `main` dentro do manifesto, é necessário adicionar o JAR ao **Classpath** e indicar a classe a ser executada:
```bash
java -classpath myjar.jar HelloWorld
# ou de forma abreviada:
java -cp myjar.jar HelloWorld
```

### 4.3. Estrutura interna de um JAR
Um arquivo `.jar` é um arquivo ZIP convencional. Ao descompactá-lo (`unzip myjar.jar`), encontramos:
```
myjar/
├── HelloWorld.class
└── META-INF/
    └── MANIFEST.MF
```
O arquivo `META-INF/MANIFEST.MF` armazena metadados da aplicação, como a versão do manifesto e, opcionalmente, o ponto de entrada da aplicação (`Main-Class`).

---

## 5. Gerenciando Bibliotecas de Terceiros e o Classpath

Quando uma aplicação consome bibliotecas externas (como o projeto Apache Commons Lang), o compilador (`javac`) e a JVM (`java`) precisam saber onde essas classes se encontram.

### 5.1. Código utilizando `StringUtils`
```java
import org.apache.commons.lang3.StringUtils;

public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World");
        // Capitaliza apenas a primeira letra da string
        System.out.println(StringUtils.capitalize("hello world")); 
    }
}
```

### 5.2. Compilação com dependência externa
Assumindo que o arquivo JAR da biblioteca foi baixado manualmente e colocado no diretório `./lib/commons-lang3-3.x.jar`:

```bash
javac -classpath "./lib/*" HelloWorld.java
```
* O parâmetro `-classpath` (ou `-cp`) informa ao compilador onde buscar os arquivos `.class` externos importados no código.
* O curinga `"./lib/*"` carrega todos os JARs contidos na pasta `lib`.

### 5.3. Execução com dependência externa e classe local
Na execução, a JVM precisa ter acesso tanto aos JARs externos quanto à classe compilada local (`HelloWorld.class`, que está no diretório corrente `.`):

* **Linux / macOS (separador de classpath é dois-pontos `:`):**
  ```bash
  java -classpath "./lib/*:." HelloWorld
  ```

* **Windows (separador de classpath é ponto-e-vírgula `;`):**
  ```cmd
  java -classpath "./lib/*;." HelloWorld
  ```

### 5.4. A Dor Resolvida pelo Maven: "JAR Hell"
Fazer isso manualmente revela limitações graves do desenvolvimento artesanal:
1. **Transitividade de dependências:** Se a biblioteca A depender da biblioteca B, você precisa descobrir, baixar manualmente e adicionar B ao classpath.
2. **Conflito de versões:** Se duas bibliotecas precisarem de versões diferentes de uma mesma terceira dependência, o classpath carrega a primeira que encontrar, gerando erros em tempo de execução (`ClassNotFoundException`, `NoSuchMethodError`).
3. **Falta de reprodutibilidade:** Diferenças de sistema operacional (como o separador `:` vs `;`) tornam os scripts de compilação manuais frágeis e difíceis de compartilhar entre equipes e ambientes de CI/CD.

O **Apache Maven** foi criado exatamente para resolver essas dores através de convenções, repositórios centrais e resolução transitiva de dependências.

---

## 6. Apêndice — Atualizações & Boas Práticas Modernas

A transcrição do curso demonstra comandos fundamentais clássicos. No ecossistema moderno de desenvolvimento Java e Maven, destacam-se as seguintes evoluções:

### 1. Execução direta de arquivos `.java` sem compilação prévia (Java 11+)
Desde o Java 11 ([JEP 330](https://openjdk.org/jeps/330)), você não precisa rodar `javac` seguido de `java` para scripts simples. A JVM compila o arquivo em memória e o executa em um único comando:
```bash
java HelloWorld.java
```

### 2. Definição do ponto de entrada (`Main-Class`) via comando `jar`
Para criar um JAR executável via linha de comando sem precisar editar manualmente o arquivo `MANIFEST.MF`, usa-se a flag `e` (*entrypoint*):
```bash
jar cfe myjar.jar HelloWorld HelloWorld.class
```
Assim, o JAR pode ser executado diretamente com:
```bash
java -jar myjar.jar
```

### 3. Simplificação de Classes e Método Main (Java 21+)
A partir do Java 21 (como *Preview Feature* via JEP 445 e refinado nas versões subsequentes 22, 23 e 24 como *Implicitly Declared Classes and Instance Main Methods*), o código de programas simples dispensa a verbosidade do `public class` e do `public static void main(String[] args)`:
```java
void main() {
    println("Hello, World!");
}
```

### 4. Declínio de EAR e WAR em favor de Microservices & Cloud-Native
* **EAR:** Praticamente extinto em novos projetos; mantido apenas em sistemas legados corporativos.
* **WAR:** Substituído majoritariamente por **Fat JARs** auto-contidos gerados por frameworks modernos (Spring Boot, Quarkus, Micronaut).
* **Containerização sem Dockerfile:** Hoje em dia, plugins do Maven como o **Google Jib** (`jib-maven-plugin`) e o **Spring Boot Buildpack** (`mvn spring-boot:build-image`) constroem imagens OCI/Docker diretamente a partir do código compilado, sem a necessidade de manter scripts de compilação manuais ou até mesmo arquivos `Dockerfile`.
