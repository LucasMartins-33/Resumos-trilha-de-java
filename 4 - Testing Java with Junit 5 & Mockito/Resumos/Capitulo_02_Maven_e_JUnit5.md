# Capítulo 02: Adicionando JUnit 5 a um Projeto Maven

Este capítulo foca na configuração inicial de um projeto Java utilizando o **Maven** como gerenciador de dependências e na inclusão e configuração das bibliotecas do JUnit 5.

## 1. Criando um Projeto Maven
Ao criar um projeto Maven, você precisará preencher duas propriedades principais (coordenadas do artefato) que identificam sua aplicação de forma única:
*   **GroupId:** Normalmente representa o domínio reverso da empresa ou desenvolvedor (ex: `com.seunome.projeto` ou `com.appsdeveloperblog`).
*   **ArtifactId:** É o nome do projeto ou do módulo em si (ex: `CalculatorMavenProject`).

A estrutura padrão de pastas que o Maven cria (e que deve ser rigorosamente respeitada para que as ferramentas de build funcionem corretamente) é:
*   `src/main/java`: Onde fica o código-fonte principal da sua aplicação.
*   `src/test/java`: Onde devem ficar as suas classes e códigos de teste.
*   `pom.xml`: O arquivo "coração" do projeto Maven, onde configuramos bibliotecas, dependências e plugins.

## 2. Adicionando Dependências do JUnit 5
Para que o seu projeto reconheça as anotações do JUnit 5 (como `@Test`) e consiga rodar os testes, você precisa adicionar as dependências no arquivo `pom.xml`, dentro da tag `<dependencies>`.

> [!TIP]
> **Atualização importante:** Em aulas e versões mais antigas do JUnit 5, o instrutor adicionava dependências isoladas (`junit-jupiter-api`, `junit-jupiter-engine` e `junit-jupiter-params`). Hoje em dia, a prática oficial e mais recomendada é usar apenas o **agregador genérico `junit-jupiter`**, que já traz o pacote inteiro de uma vez!

### 📝 Sintaxe: Atualizando o `pom.xml` (Forma Moderna)
```xml
<dependencies>
    <!-- Dependência Agregadora do JUnit 5 (Engloba API, Engine e Params) -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>5.10.2</version> <!-- Utilize sempre as versões estáveis mais atuais -->
        <scope>test</scope>
    </dependency>
</dependencies>
```
*Dica: Lembre-se de sempre clicar no botão de "Reload" (Recarregar projeto Maven) na sua IDE após alterar o `pom.xml` para que ela faça o download das bibliotecas do repositório.*

## 3. Rodando Testes pelo Terminal e o Maven Surefire Plugin
Embora seja prático rodar testes clicando no "Play" da IDE (IntelliJ / Eclipse), no dia a dia e em ambientes profissionais (como esteiras de CI/CD), usamos o terminal e os comandos do Maven.

**Comandos básicos de terminal:**
*   `mvn test`: Executa estritamente os testes unitários do projeto.
*   `mvn package`: Compila o código, executa os testes unitários e depois empacota a aplicação num arquivo `.jar`.

### O Maven Surefire Plugin
Para que os comandos do Maven consigam encontrar as classes na pasta `/test` e executá-las usando o JUnit 5, o Maven faz o uso de um plugin chamado **Maven Surefire Plugin**.

> [!NOTE]
> **Atualização e Dica Prática:** Em projetos Java modernos que usam o **Spring Boot** ou que possuam o Maven atualizado (Surefire Plugin na versão `3.0.0` ou superior), o suporte ao JUnit 5 **já é nativo e vem pré-configurado**. Nesses casos, você não precisa fazer nada!

Caso o seu projeto seja Java "puro" e antigo, e os testes não estejam sendo detectados pelo terminal, você precisa adicionar o plugin explicitamente dentro da tag `<build>`:

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>3.2.5</version> <!-- Versão moderna recomendada -->
        </plugin>
    </plugins>
</build>
```

### Como pular os testes no build (Skip Tests)
Se por algum motivo excepcional você precisar empacotar o projeto ou gerar um build ignorando as falhas de teste (ou pulando eles para ganhar tempo), você pode usar a *flag* `maven.test.skip`:

```bash
# Comando para empacotar a aplicação sem executar a bateria de testes:
mvn package -Dmaven.test.skip=true
```
*(Cuidado: Essa prática anula a vantagem de se ter testes de segurança, use com sabedoria!)*
