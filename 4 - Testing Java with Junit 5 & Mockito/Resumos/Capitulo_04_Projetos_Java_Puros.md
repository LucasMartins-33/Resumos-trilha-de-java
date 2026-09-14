# Capítulo 04: Adicionando JUnit 5 a Projetos Java Básicos (Sem Maven/Gradle)

Apesar de Maven e Gradle serem o padrão da indústria para gerenciamento de dependências e builds complexos, é totalmente possível configurar o JUnit 5 de forma manual em projetos Java "puros" ou acadêmicos utilizando os atalhos e assistentes das próprias IDEs.

## 1. Configurando no IntelliJ IDEA

Em um projeto Java criado de forma simples, você precisará configurar as pastas e baixar a biblioteca manualmente através de cliques na IDE.

**Passo a passo:**
1.  **Criando e Mapeando a pasta de testes:** 
    *   Na raiz do seu projeto, crie um novo diretório chamado `test`.
    *   Clique com o botão direito na pasta `test` -> Vá em **Mark Directory as** -> Selecione **Test Sources Root**. 
    *   *Nota: A pasta mudará de cor (geralmente fica verde), indicando ao IntelliJ que ali moram os testes da aplicação.*
2.  **Gerando a classe de teste automaticamente:**
    *   Abra a classe de produção (ex: `Calculator`).
    *   Use o atalho (ou clique com botão direito) -> **Generate** -> **Test...**.
    *   Selecione **JUnit 5** como a biblioteca alvo.
    *   **O Truque:** A IDE mostrará um aviso (lâmpada amarela ou alerta) dizendo *"JUnit 5 library not found in the module"*. Basta clicar no botão **Fix** e o próprio IntelliJ fará o download do arquivo `.jar` do JUnit 5 e o adicionará ao *classpath* da sua aplicação automaticamente.
    *   *Boa prática:* O pacote (`package`) da classe de teste deve ser idêntico ao pacote da classe que está sendo testada.
3.  **Comportamento padrão dos Testes:** Se um método anotado com `@Test` terminar sua execução sem falhar em nenhuma validação e sem lançar exceções, o JUnit assumirá que **o teste passou (Verde)**, mesmo que ele esteja vazio!
4.  **Forçando Falhas (Fail Fast):** Se você quiser forçar um erro para ver o teste ficar vermelho, você pode usar a asserção `fail()`:
    ```java
    import static org.junit.jupiter.api.Assertions.fail;

    @Test
    void testeAindaNaoImplementado() {
        fail("Falha proposital: Este teste ainda precisa ser escrito!");
    }
    ```

## 2. Configurando no Eclipse IDE

Se você utilizar o Eclipse, o processo é bem semelhante e também guiado por assistentes.

**Passo a passo:**
1.  **Criando a pasta (Source Folder):**
    *   Na raiz do projeto, clique com botão direito -> **New** -> **Source Folder**. Nomeie como `test`. (Tem que ser *Source Folder* e não um folder comum).
2.  **Gerando o Caso de Teste (Test Case):**
    *   Clique na sua classe Java (ex: `Calculator`) com botão direito -> **New** -> **JUnit Test Case**.
    *   Lá em cima, marque a opção **"New JUnit Jupiter test"** (Isso garante que é o JUnit 5 e não versões velhas).
    *   Aponte o campo de destino *Source folder* para a pasta `test` que você criou.
3.  **Adicionando ao Build Path:**
    *   Ao concluir o wizard, o Eclipse vai reclamar que o JUnit 5 não está no *Build Path*. 
    *   Vai aparecer uma janela com a opção **"Perform the following action"**. Deixe marcado, dê `OK`, e o Eclipse conectará os arquivos `.jar` do JUnit ao seu projeto.
4.  **Executando:** 
    *   Clique com botão direito na classe ou arquivo -> **Run As** -> **JUnit Test**. 
    *   O painel do JUnit abrirá na lateral com a icônica **Barra Verde** (sucesso) ou **Barra Vermelha** (falha).
