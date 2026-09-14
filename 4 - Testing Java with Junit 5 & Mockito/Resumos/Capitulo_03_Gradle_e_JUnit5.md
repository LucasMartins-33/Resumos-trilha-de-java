# Capítulo 03: Adicionando JUnit 5 a um Projeto Gradle

Neste capítulo, aprendemos a criar e configurar um projeto Java utilizando o **Gradle** como ferramenta de automação de build e gerenciamento de dependências, configurando corretamente o suporte ao JUnit 5.

## 1. Criando um Projeto Gradle
Ao criar um projeto com Gradle (assim como ocorre no Maven), você precisará fornecer as coordenadas de identificação do artefato:
*   **GroupId:** O domínio reverso do seu projeto, empresa ou seu nome (ex: `com.appsdeveloperblog` ou `com.lucas`).
*   **ArtifactId:** O nome específico do projeto (ex: `CalculatorGradleProject`).

A estrutura de diretórios gerada é padronizada e deve ser mantida:
*   `src/main/java`: Para o código-fonte principal da aplicação.
*   `src/test/java`: Para o código dos testes unitários.
*   `build.gradle`: É o arquivo principal de configuração de build e dependências (o equivalente ao `pom.xml` do Maven). Pode ser escrito em Groovy ou Kotlin DSL.

## 2. Configurando Dependências no `build.gradle`
Para utilizar o JUnit 5 em um projeto Gradle, você deve declarar a dependência na seção `dependencies` do arquivo `build.gradle` e avisar ao Gradle para usar a plataforma correta na seção `test`.

> [!TIP]
> Assim como abordado no capítulo sobre Maven, o instrutor mostra a busca por pacotes separados, mas a melhor prática moderna é buscar pelo pacote agregador `junit-jupiter`, que baixa a API, a Engine e a biblioteca de parâmetros de uma só vez.

### 📝 Sintaxe: Atualizando o `build.gradle` (em Groovy)
```groovy
dependencies {
    // Declara a dependência agregadora moderna do JUnit 5 para o escopo de teste
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.2'
}

// Configuração obrigatória da task de testes do Gradle
test {
    // Informa ao Gradle para usar a infraestrutura do JUnit Platform (JUnit 5)
    useJUnitPlatform()
    
    // (Opcional) Configuração extra útil ensinada na aula:
    // Faz com que os prints (System.out) gerados dentro dos testes apareçam no console do terminal.
    testLogging {
        showStandardStreams = true 
    }
}
```
*Dica: Após qualquer alteração no `build.gradle`, lembre-se de clicar no botão "Load/Reload Gradle Changes" que aparece no canto superior direito do IntelliJ.*

## 3. Rodando Testes pelo Terminal e o Gradle Wrapper
Além da execução tradicional pelo botão de "Play" ao lado do método na IDE, é fundamental saber rodar os testes via linha de comando. 

Em projetos Gradle, geralmente utilizamos o **Gradle Wrapper** (arquivos `gradlew` e `gradlew.bat` que vêm na raiz do projeto). Ele garante que o projeto será executado na mesma versão do Gradle em qualquer máquina, sem precisar que você instale o Gradle globalmente no computador.

**Comandos básicos no terminal:**
```bash
# Executando no Mac/Linux:
./gradlew clean test

# Executando no Windows (Powershell / CMD):
.\gradlew clean test
```
**O que este comando faz:**
1.  `clean`: Apaga a pasta de build gerada anteriormente, garantindo que o projeto será recompilado do zero (evita bugs de cache).
2.  `test`: Executa a tarefa de teste, que vasculhará a pasta `src/test/java` executando tudo que estiver anotado com `@Test`.
