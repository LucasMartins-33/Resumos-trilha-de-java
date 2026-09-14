# Questões Práticas - Capítulo 09 (Cobertura de Código - Code Coverage)

---

### 🟢 Nível 1: Executando com Cobertura no IntelliJ IDEA
**Cenário:** Você tem uma classe utilitária e quer descobrir rapidamente quais métodos dela já possuem testes unitários ativos sem sair do editor.
**Sua Tarefa:**
* Clique com o botão direito na sua classe de teste e selecione a opção **Run with Coverage** (ícone do escudo com play).
* Observe a abertura da aba lateral de cobertura (*Coverage Tool Window*).
* Localize as porcentagens de cobertura de Classe (% Class), Métodos (% Method) e Linhas (% Line).

---

### 🟡 Nível 2: Inspecionando Marcadores Coloridos no Editor de Código
**Cenário:** Ao abrir o arquivo de produção após rodar a cobertura no Nível 1, você nota barras coloridas ao lado dos números de linha.
**Sua Tarefa:**
* Abra a classe de produção no editor.
* Localize uma linha com a **Barra Verde** e explique o que ela significa.
* Localize uma linha com a **Barra Vermelha** e escreva um novo método de teste unitário especificamente para passar por aquela linha não executada.
* Reexecute a cobertura e certifique-se de que a barra vermelha se transformou em verde.

---

### 🟠 Nível 3: Configurando o Plugin JaCoCo no Maven
**Cenário:** A equipe precisa que a cobertura de código seja calculada automaticamente durante as builds do Maven e gere um relatório HTML navegável.
**Sua Tarefa:**
* No arquivo `pom.xml`, adicione o plugin `jacoco-maven-plugin` dentro de `<build><plugins>`.
* Configure as duas execuções essenciais:
  1. `prepare-agent`: vinculada à fase de inicialização para acoplar o agente de rastreamento.
  2. `report`: vinculada à fase `test` para compilar o relatório após a execução.

---

### 🔴 Nível 4: Gerando e Navegando pelo Relatório HTML do JaCoCo
**Cenário:** Você precisa disponibilizar o relatório de cobertura gerado pelo Maven para a revisão técnica da equipe.
**Sua Tarefa:**
* Execute no terminal o comando `mvn clean test`.
* Navegue até o diretório `target/site/jacoco/`.
* Abra o arquivo `index.html` em seu navegador.
* Clique nos links navegáveis para descer do nível de pacote -> classe -> métodos.

---

### 🟣 Nível 5: Diagnosticando a Métrica "Missed Instructions"
**Cenário:** O relatório do JaCoCo aponta que sua classe possui 100% de linhas cobertas, mas relata instruções perdidas (*Missed Instructions*).
**Sua Tarefa:**
* Analise um método contendo operadores ternários ou atribuições em linha única (ex: `int max = (a > b) ? a : b;`).
* Explique por que uma linha física pode conter múltiplas instruções de bytecode compiladas.
* Escreva os testes necessários para garantir que ambos os lados da atribuição sejam executados.

---

### 🟤 Nível 6: Interpretando e Resolvendo o "Losango Amarelo" (Missed Branches)
**Cenário:** No relatório HTML do JaCoCo, uma estrutura `if (idade >= 18 && temCarteira)` está marcada com um losango amarelo 🟡 ao lado da linha.
**Sua Tarefa:**
* O que o losango amarelo indica em termos de ramificações lógicas (*branches*)?
* Escreva testes unitários para cobrir as combinações faltantes:
  * Cenário 1: idade >= 18 e temCarteira = true.
  * Cenário 2: idade >= 18 e temCarteira = false.
  * Cenário 3: idade < 18.
* Reexecute `mvn test` e comprove que o losango se tornou verde 🟢.

---

### 🔵 Nível 7: Identificando o "Losango Vermelho" em Blocos Condicionais
**Cenário:** Uma cláusula de validação defensiva `if (parametro == null) { throw new IllegalArgumentException(); }` está marcada com um losango vermelho 🔴.
**Sua Tarefa:**
* Explique por que o losango vermelho indica que nenhuma das decisões foi testada (ou todo o bloco de desvio foi ignorado).
* Escreva um teste passando `null` e validando o lançamento da exceção com `assertThrows`.
* Verifique a atualização da métrica no JaCoCo.

---

### 🟢 Nível 8: Avaliando a Complexidade Ciclomática (Cyclomatic Complexity)
**Cenário:** Ao analisar o relatório JaCoCo, você percebe que um método `processarPagamento` tem Complexidade Ciclomática igual a 12 (Cxty = 12), exibindo uma grande barra vermelha de complexidade.
**Sua Tarefa:**
* Explique por que uma alta complexidade ciclomática indica que o método possui muitos caminhos e desvios lógicos independentes.
* O que acontece com a quantidade mínima de testes unitários necessários para cobrir esse método?
* Refatore o método no código de produção: divida-o em submétodos menores com responsabilidades únicas e observe a redução do índice de complexidade no relatório.

---

### 🟡 Nível 9: Relatório de Execução de Testes com o Surefire Report Plugin
**Cenário:** Além da cobertura, você quer gerar um documento HTML consolidando todos os testes executados e seus tempos (`surefire-report.html`).
**Sua Tarefa:**
* Adicione o `maven-surefire-report-plugin` ao `pom.xml`.
* Configure `<testFailureIgnore>true</testFailureIgnore>` no `maven-surefire-plugin` principal para que a compilação do relatório ocorra mesmo se houver falhas.
* Execute `mvn test surefire-report:report` e inspecione o arquivo gerado em `target/site/surefire-report.html`.

---

### 🟠 Nível 10: Integração Final (Atingindo a Meta Saudável de Cobertura)
**Cenário:** A política de qualidade da sua empresa exige uma meta mínima de **80% de cobertura de linhas e ramificações** para aprovação do pull request.
**Sua Tarefa:**
* Crie a classe `CalculadoraFrete.java` com regras condicionais para diferentes regiões (SUL, SUDESTE, NORTE, NORDESTE), limites de peso e cupons de frete grátis.
* Execute o JaCoCo com sua primeira bateria de testes e observe a porcentagem inicial (ex: 45%).
* Adicione incrementalmente novos cenários de teste unitário focando nos pontos cegos (linhas vermelhas e losangos amarelos apontados pelo relatório).
* Reexecute `mvn clean test` e comprove que a suíte superou a meta de 80% em `target/site/jacoco/index.html`.
