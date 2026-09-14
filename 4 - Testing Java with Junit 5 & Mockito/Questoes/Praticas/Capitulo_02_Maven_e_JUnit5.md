# Questões Práticas - Capítulo 02 (Adicionando JUnit 5 a um Projeto Maven)

---

### 🟢 Nível 1: Definindo as Coordenadas Maven (GAV)
**Cenário:** Você está iniciando um projeto novo para o módulo de pagamentos da sua empresa chamada "FinanceHub" e precisa definir as coordenadas básicas no arquivo `pom.xml`.
**Sua Tarefa:**
* Escreva um arquivo `pom.xml` inicial definindo:
  * `<groupId>` com o domínio invertido: `com.financehub.billing`.
  * `<artifactId>` com o nome do artefato: `payment-service`.
  * `<version>`: `1.0.0-SNAPSHOT`.
  * Versão do Java configurada nas propriedades (`maven.compiler.source` e `target` para 17 ou 21).

---

### 🟡 Nível 2: Estruturando o Diretório Padrão
**Cenário:** Um estagiário colocou classes Java em diretórios aleatórios e o Maven não consegue compilar o projeto.
**Sua Tarefa:**
* Organize a estrutura de pastas física rigorosamente segundo a convenção do Maven:
  * Onde deve ficar a classe de produção `CalculadoraFinanceira.java`?
  * Onde deve ficar a classe de teste `CalculadoraFinanceiraTest.java`?
  * Onde deve ficar o arquivo `pom.xml`?
* Descreva a hierarquia de pastas completa em formato de árvore de diretórios.

---

### 🟠 Nível 3: Adicionando a Dependência Agregadora do JUnit 5
**Cenário:** Você precisa importar o JUnit 5 para o projeto usando a melhor prática moderna recomendada (sem dependências fragmentadas).
**Sua Tarefa:**
* Adicione o bloco `<dependencies>` no seu `pom.xml`.
* Declare o artefato agregador `org.junit.jupiter:junit-jupiter` na versão `5.10.2`.
* Garanta a configuração do escopo correto `<scope>test</scope>`.

---

### 🔴 Nível 4: Diagnosticando o Impacto do Escopo Incorreto
**Cenário:** Um colega removeu acidentalmente a tag `<scope>test</scope>` da dependência do JUnit no `pom.xml`.
```xml
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.2</version>
    <!-- Tag scope removida! -->
</dependency>
```
**Sua Tarefa:**
* Explique o que acontece com a visibilidade das anotações `@Test` e `Assertions` dentro das classes em `src/main/java`.
* O que acontecerá com o arquivo `.jar` gerado para produção se o projeto for compilado com `mvn package` sem a tag `<scope>test</scope>`?
* Corrija o bloco inserindo o escopo apropriado.

---

### 🟣 Nível 5: Executando Testes via Terminal com `mvn test`
**Cenário:** Você precisa executar a bateria de testes unitários diretamente pela linha de comando, sem abrir a interface gráfica da IDE.
**Sua Tarefa:**
* Abra o terminal na raiz do projeto onde reside o `pom.xml`.
* Execute o comando `mvn test`.
* Identifique na saída do console onde o Maven imprime o resumo da execução:
  * Total de testes executados (*Tests run*).
  * Falhas (*Failures*).
  * Erros (*Errors*).
  * Testes ignorados (*Skipped*).

---

### 🟤 Nível 6: Compilação Completa e Empacotamento com `mvn package`
**Cenário:** Você precisa gerar o artefato executável da aplicação para envio ao ambiente de homologação.
**Sua Tarefa:**
* Execute o comando `mvn package` no terminal.
* Observe o ciclo de vida do Maven: verifique se a fase `test` é executada automaticamente antes da fase de geração do `.jar`.
* Caso um dos seus testes unitários em `src/test/java` falhe intencionalmente com `fail("Erro forçado")`, o arquivo `.jar` na pasta `target/` é gerado? Comprove observando o encerramento do build com `BUILD FAILURE`.

---

### 🔵 Nível 7: Configurando o Maven Surefire Plugin
**Cenário:** Em um projeto Java legado sem Spring Boot, ao rodar `mvn test` o console reporta `0 tests executed`, embora existam métodos anotados com `@Test` do JUnit 5.
**Sua Tarefa:**
* Adicione a tag `<build><plugins>` no seu `pom.xml`.
* Configure o `maven-surefire-plugin` na versão estável `3.2.5` com groupId `org.apache.maven.plugins`.
* Reexecute `mvn test` e valide que os testes passam a ser descobertos pela engine do Surefire.

---

### 🟢 Nível 8: Pulando Testes com Segurança usando Flags do Maven
**Cenário:** Em uma situação emergencial de build local em que um teste de integração de rede está oscilando, você precisa gerar o pacote `.jar` rapidamente sem executar os testes.
**Sua Tarefa:**
* Escreva e execute o comando Maven que utiliza a propriedade `maven.test.skip` definida como `true`.
* Verifique se o Maven gerou o `.jar` na pasta `target/` pulando a fase de compilação e execução de testes.
* Adicione um comentário na documentação da sua equipe alertando sobre o perigo de utilizar esse comando em servidores de CI/CD.

---

### 🟡 Nível 9: Recarregando o Modelo Maven na IDE (Sync / Reload)
**Cenário:** Você atualizou a versão do `junit-jupiter` no `pom.xml`, mas o IntelliJ IDEA continua sublinhando a palavra `@Test` em vermelho com a mensagem "Cannot resolve symbol 'Test'".
**Sua Tarefa:**
* Identifique por que a edição direta de texto no `pom.xml` não reflete instantaneamente no classpath do editor.
* Execute a ação necessária na sua IDE (atalho de *Reload All Maven Projects* ou clique no ícone flutuante do elefante/M).
* Verifique se o módulo baixa os novos JARs para o repositório local `~/.m2/repository`.

---

### 🟠 Nível 10: Integração Final (Pipeline de Build Completo com Maven)
**Cenário:** Você está configurando um novo repositório corporativo e precisa entregar uma aplicação com uma classe de negócio e seu respectivo teste unitário totalmente validados pelo Maven.
**Sua Tarefa:**
* Crie uma classe `ValidadorEmail.java` em `src/main/java/com/financehub/billing` com o método `public boolean isEmailValido(String email)` (retorna `true` se contiver `"@"` e `"."`).
* Crie a classe `ValidadorEmailTest.java` em `src/test/java/com/financehub/billing` contendo testes para e-mails válidos e inválidos.
* No terminal, execute `mvn clean package`.
* Certifique-se de que o build finaliza com `BUILD SUCCESS`, que todos os testes passaram e que o arquivo `payment-service-1.0.0-SNAPSHOT.jar` foi criado dentro de `target/`.
