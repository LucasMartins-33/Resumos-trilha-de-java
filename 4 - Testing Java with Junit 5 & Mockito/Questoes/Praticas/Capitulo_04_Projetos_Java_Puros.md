# Questões Práticas - Capítulo 04 (Adicionando JUnit 5 a Projetos Java Básicos)

---

### 🟢 Nível 1: Mapeando a Pasta de Testes no IntelliJ IDEA
**Cenário:** Em um projeto Java puro criado sem Maven ou Gradle, você precisa criar um diretório separado para os testes e avisar à IDE sobre sua finalidade.
**Sua Tarefa:**
* Crie uma pasta chamada `test` na raiz do projeto.
* Clique com o botão direito na pasta `test` -> selecione **Mark Directory as** -> clique em **Test Sources Root**.
* Observe a alteração visual da cor do diretório na árvore de arquivos (geralmente passa a ter um ícone verde).

---

### 🟡 Nível 2: O Atalho Mágico de Geração e o Botão "Fix"
**Cenário:** Você tem uma classe `StringUtils` e deseja gerar sua classe de testes sem precisar escrever o arquivo `.java` manualmente.
**Sua Tarefa:**
* Abra a classe `StringUtils` no editor de código.
* Pressione o atalho de geração (Alt + Insert no Windows/Linux ou Cmd + N no macOS) e escolha **Test...**.
* Selecione **JUnit 5** como a biblioteca alvo.
* Ao notar o alerta *"JUnit 5 library not found in the module"*, clique no botão **Fix**.
* Verifique que o IntelliJ baixa os arquivos JAR do JUnit 5 e os anexa automaticamente às dependências do módulo.

---

### 🟠 Nível 3: Mapeando no Eclipse (Source Folder vs Folder Comum)
**Cenário:** Um colega criou uma pasta comum chamada `test` no Eclipse e não consegue compilar classes Java dentro dela.
**Sua Tarefa:**
* Explique a diferença no Eclipse entre uma pasta simples (*Folder*) e uma pasta de código (*Source Folder*).
* Crie uma **Source Folder** chamada `test`.
* Clique na classe com o botão direito -> selecione **New** -> **JUnit Test Case**.
* Certifique-se de marcar a opção **New JUnit Jupiter test** e selecione a pasta `test` como destino.
* Permita que o assistente adicione a biblioteca do JUnit 5 ao *Build Path*.

---

### 🔴 Nível 4: Investigando o Comportamento do Teste Vazio
**Cenário:** Um desenvolvedor criou um método de teste anotado com `@Test`, mas não escreveu nenhuma linha de código dentro dele.
```java
@Test
void testeMetodoAindaVazio() {
    // Vazio propositalmente
}
```
**Sua Tarefa:**
* Execute esse método na sua IDE.
* O teste fica verde ou vermelho? Explique por que o JUnit 5 assume sucesso quando nenhuma exceção é disparada.
* Reflita sobre o perigo desse comportamento padrão em baterias de testes com regras pendentes de implementação.

---

### 🟣 Nível 5: Forçando Falhas com `Assertions.fail()`
**Cenário:** Para evitar que testes incompletos passem em branco (como visto no Nível 4), você quer marcar explicitamente os testes pendentes como falhos.
**Sua Tarefa:**
* Importe estaticamente o método `fail` de `org.junit.jupiter.api.Assertions`.
* Adicione a instrução `fail("Teste ainda não implementado! Favor codificar antes da release.")` no corpo do teste.
* Execute o teste e confirme que a barra de execução fica vermelha exibindo a mensagem fornecida.

---

### 🟤 Nível 6: Espelhamento Rigoroso de Pacotes
**Cenário:** Sua classe de produção está no pacote `com.empresa.utilidades` dentro de `src`.
**Sua Tarefa:**
* Crie a classe de teste correspondente dentro da pasta `test`.
* Declare no topo do arquivo de teste a instrução `package com.empresa.utilidades;`.
* Crie na classe de produção um método auxiliar com visibilidade padrão (*package-private*, sem palavra-chave `public`).
* Verifique se o teste consegue invocar diretamente o método auxiliar devido ao compartilhamento de pacote.

---

### 🔵 Nível 7: Executando Testes e Interpretando a Barra de Status
**Cenário:** Você precisa executar múltiplos testes e interpretar os painéis de resultados das IDEs.
**Sua Tarefa:**
* Crie dois métodos na classe de teste: um com asserção bem-sucedida (`assertEquals(10, 5 + 5)`) e outro com falha proposital (`assertEquals(10, 2 * 3)`).
* Execute a classe inteira clicando no botão "Play" ao lado do nome da classe de teste.
* No painel do JUnit da IDE, observe a **Barra Vermelha**, a lista com o teste que passou (ícone verde) e o teste que falhou (ícone vermelho acompanhado do *Comparison Failure*).

---

### 🟢 Nível 8: Entendendo a Visibilidade dos Métodos no JUnit 5
**Cenário:** Um desenvolvedor acostumado com JUnit 4 declarou todos os métodos e classes de teste como `public class` e `public void`.
**Sua Tarefa:**
* Remova a palavra-chave `public` tanto da declaração da classe de teste quanto dos métodos anotados com `@Test`.
* Execute o teste e comprove que o JUnit 5 roda perfeitamente sem exigir modificadores públicos.
* Explique a vantagem dessa mudança arquitetural do JUnit 5 em relação ao JUnit 4.

---

### 🟡 Nível 9: Diagnosticando as Limitações em Ambientes Headless (CI/CD)
**Cenário:** A equipe quer colocar o projeto Java puro em uma esteira de integração contínua (GitHub Actions).
**Sua Tarefa:**
* Explique por que um projeto cujas dependências de teste foram configuradas apenas via assistente gráfico da IDE (sem `pom.xml` ou `build.gradle`) enfrenta graves problemas para rodar em servidores headless de CI/CD.
* O que seria necessário fazer para compilar e executar manualmente os JARs via linha de comando no terminal puro usando `javac` e `java -jar` da JUnit Platform Console Launcher?

---

### 🟠 Nível 10: Integração Final (Classe Utilitária Testada Manualmente)
**Cenário:** Em um projeto sem gerenciadores de dependência, você precisa construir uma classe de formatação e sua suíte de testes completa rodando na IDE.
**Sua Tarefa:**
* Crie a classe `FormatadorTexto.java` em `src/com/empresa/texto`:
  * Método `removerEspacosExtras(String texto)`: remove espaços no início, fim e espaços duplicados no meio.
* Na pasta `test/com/empresa/texto`, crie `FormatadorTextoTest.java`.
* Escreva testes unitários cobrindo:
  1. String com espaços no início e fim (`"  teste  "` -> `"teste"`).
  2. String com múltiplos espaços internos (`"Olá    Mundo"` -> `"Olá Mundo"`).
  3. String nula ou vazia.
* Execute a suíte completa pela IDE e garanta a icônica **Barra Verde** com 100% de aprovação.
