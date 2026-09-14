# Questões Práticas - Capítulo 03 (Adicionando JUnit 5 a um Projeto Gradle)

---

### 🟢 Nível 1: Estruturando o Arquivo `build.gradle`
**Cenário:** Você está iniciando um projeto Java moderno que utilizará o Gradle em vez do Maven para automação de compilação.
**Sua Tarefa:**
* Crie o arquivo `build.gradle` (DSL Groovy).
* Aplique o plugin do Java: `plugins { id 'java' }`.
* Configure o repositório Maven Central: `repositories { mavenCentral() }`.
* Defina as propriedades `group = 'com.meuprojeto'` e `version = '1.0.0'`.

---

### 🟡 Nível 2: Declarando a Dependência do JUnit 5 no Gradle
**Cenário:** Você precisa importar as bibliotecas do JUnit Jupiter para a execução de testes automatizados.
**Sua Tarefa:**
* No bloco `dependencies` do `build.gradle`, adicione o artefato agregador do JUnit 5 utilizando a diretiva `testImplementation`:
  ```groovy
  dependencies {
      testImplementation 'org.junit.jupiter:junit-jupiter:5.10.2'
  }
  ```
* Explique por que a diretiva `testImplementation` é preferível a `implementation` para bibliotecas de testes.

---

### 🟠 Nível 3: A Configuração Obrigatória `useJUnitPlatform()`
**Cenário:** Você escreveu testes com `@Test` do JUnit 5, mas ao rodar o Gradle pelo terminal, o log informa que nenhum teste foi executado (*0 tests completed*).
**Sua Tarefa:**
* Adicione o bloco de configuração da task `test` no `build.gradle`:
  ```groovy
  test {
      useJUnitPlatform()
  }
  ```
* Explique a razão técnica pela qual essa instrução é estritamente obrigatória para o JUnit 5 no Gradle.

---

### 🔴 Nível 4: Habilitando Prints de Console nos Testes (`showStandardStreams`)
**Cenário:** Durante a depuração de um teste no terminal, você adicionou comandos `System.out.println()`, mas o Gradle oculta a saída do console por padrão.
**Sua Tarefa:**
* Modifique o bloco `test` no `build.gradle` para habilitar a exibição de fluxos de saída padrão:
  ```groovy
  test {
      useJUnitPlatform()
      testLogging {
          showStandardStreams = true
      }
  }
  ```
* Crie um método de teste com `System.out.println("Executando teste no Gradle...")` e comprove a exibição do print no terminal.

---

### 🟣 Nível 5: Executando Testes via Gradle Wrapper no Terminal
**Cenário:** Você precisa rodar a suíte de testes em um ambiente Unix/Linux ou Windows garantindo o uso da versão exata do Gradle fornecida pelo projeto.
**Sua Tarefa:**
* Identifique os scripts do Gradle Wrapper na raiz do projeto (`gradlew` para Linux/macOS e `gradlew.bat` para Windows).
* No terminal Linux/macOS, execute os testes com `./gradlew test`.
* No terminal Windows (PowerShell), execute os testes com `.\gradlew test`.
* Explique por que utilizamos o prefixo `./` em ambientes Unix.

---

### 🟤 Nível 6: Limpando Caches e Forçando Reexecução com `clean test`
**Cenário:** Você fez alterações nos testes, mas o Gradle reporta que a task `test` está `UP-TO-DATE` e não reexecuta os testes devido ao mecanismo de cache incremental.
**Sua Tarefa:**
* Execute no terminal o comando combinado `./gradlew clean test`.
* Observe que o diretório `build/` é excluído antes da compilação.
* Valide que todos os testes são recompilados e executados do zero, garantindo feedback limpo.

---

### 🔵 Nível 7: Inspecionando o Relatório HTML Nativo do Gradle
**Cenário:** O gerente de qualidade quer ver um relatório gráfico detalhado com o histórico e duração dos testes executados pelo Gradle.
**Sua Tarefa:**
* Após rodar `./gradlew test`, navegue até a pasta `build/reports/tests/test/`.
* Abra o arquivo `index.html` em um navegador de internet.
* Identifique no relatório: a taxa de sucesso (ex: 100%), o tempo total de execução e o detalhamento por classes e métodos.

---

### 🟢 Nível 8: Investigando Falhas de Teste no Terminal
**Cenário:** Um dos métodos de teste falhou propositalmente durante o build do Gradle.
**Sua Tarefa:**
* Crie um teste `testFalhaProposital()` com `fail("Verificando relatório de erro")`.
* Execute `./gradlew test`.
* Observe a saída do terminal: analise o stack trace exibido e localize o link direto que o Gradle imprime no console apontando para a página HTML do teste reprovado.

---

### 🟡 Nível 9: Atualizando e Sincronizando o Gradle na IDE
**Cenário:** Você alterou uma versão de dependência no arquivo `build.gradle`, mas a IDE ainda não reconhece as novas classes.
**Sua Tarefa:**
* Localize o painel lateral do Gradle na sua IDE (IntelliJ ou Eclipse).
* Acione o botão "Reload All Gradle Projects" (ou atalho de sincronização).
* Verifique na aba *External Libraries* do projeto se as bibliotecas do JUnit Jupiter foram indexadas corretamente.

---

### 🟠 Nível 10: Integração Final (Projeto Java Completo com Gradle)
**Cenário:** Você precisa entregar uma estrutura completa de projeto automatizada via Gradle, contendo uma classe de negócios, testes unitários e build validado.
**Sua Tarefa:**
* Crie a classe `ConversorTemperatura.java` em `src/main/java` com o método `celsiusParaFahrenheit(double celsius)` (fórmula: `(celsius * 9/5) + 32`).
* Crie a classe `ConversorTemperaturaTest.java` em `src/test/java` testando os cenários: ponto de congelamento (0°C -> 32°F) e ponto de ebulição (100°C -> 212°F).
* Execute `./gradlew clean test`.
* Confirme a aprovação do build (`BUILD SUCCESSFUL`) e verifique a integridade do relatório em `build/reports/tests/test/index.html`.
