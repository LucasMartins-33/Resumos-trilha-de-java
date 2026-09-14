# Capítulo 09: Cobertura de Código (Code Coverage)

Este capítulo aborda a geração e a interpretação de relatórios de **Cobertura de Código**, uma métrica vital que ajuda a identificar quais trechos da aplicação ainda não possuem testes escritos.

## 1. Introdução à Cobertura de Código
A métrica de cobertura indica a porcentagem de classes, métodos e linhas de código que foram invocados durante a execução da sua suíte de testes.
*   **Identificando Pontos Cegos:** Se uma classe de serviço importante está com 0% de cobertura, é um risco gigante para o software. O relatório diz exatamente onde você deve focar seus próximos esforços de teste.
*   **O Cuidado com os 100%:** Atingir 100% de cobertura não significa que o software está livre de bugs (bug-free). Significa apenas que o interpretador do Java passou por aquela linha pelo menos uma vez. A qualidade da asserção (`assertEquals`) e testes com cenários negativos ainda são responsabilidade do desenvolvedor.
*   **O Padrão Ouro:** Como atingir 100% é muito raro e custoso, o mercado adota uma meta saudável entre **70% a 80%** de cobertura para considerar um projeto aprovado.

## 2. Gerando Cobertura na IDE (IntelliJ)
Para analisar a cobertura enquanto você programa, utilize o recurso nativo do IntelliJ.
1. Em vez de dar o clássico clique no ícone verde de "Play", escolha a opção **Run with Coverage** (cujo ícone é um escudo verde com o play).
2. A IDE abrirá um painel à direita mostrando a porcentagem testada de cada pacote, classe, métodos e linhas.
3. No código-fonte (lado esquerdo, junto à numeração das linhas), barras aparecerão:
   *   **Barra Verde:** A linha foi coberta pelo teste.
   *   **Barra Vermelha:** Nenhum teste passou por essa linha.

## 3. Relatórios HTML no Maven (Sem Cobertura)
Se você deseja gerar um relatório consolidado com o total de testes executados, quais passaram e quais falharam (muito útil em processos de CI/CD), adicione o plugin do Surefire ao `pom.xml`:
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-report-plugin</artifactId>
    <version>3.0.0-M5</version> <!-- Usar versão atual -->
    <executions>
        <execution>
            <phase>test</phase>
            <goals><goal>report</goal></goals>
        </execution>
    </executions>
</plugin>
```
> [!NOTE]
> Por padrão, se um teste quebra, o Maven para a execução e não gera o relatório. Para consertar isso, vá nas configurações do `maven-surefire-plugin` principal e adicione a tag `<testFailureIgnore>true</testFailureIgnore>`.

O HTML final com os resultados ficará disponível na pasta: `target/site/surefire-report.html`.

## 4. O Padrão da Indústria: JaCoCo (Java Code Coverage)
Para extrair um relatório de **cobertura real em formato HTML** através do Maven, usamos o plugin **JaCoCo**.

Adicione ao `pom.xml`:
```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.8</version> <!-- Usar versão atual -->
    <executions>
        <!-- 1. Executa o Agente antes dos testes rodarem para rastrear as linhas -->
        <execution>
            <id>prepare-agent</id>
            <goals><goal>prepare-agent</goal></goals>
        </execution>
        <!-- 2. Após a fase de testes, coleta os dados e cria o HTML -->
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals><goal>report</goal></goals>
        </execution>
    </executions>
</plugin>
```
Rodando um comando `mvn clean test`, o plugin criará um site completo na pasta `target/site/jacoco/index.html`.

### Como ler as Métricas do JaCoCo
O relatório do JaCoCo traz as seguintes estatísticas:
*   **Missed Instructions:** Mostra o percentual de bytes de instruções do Java que ficaram sem testes.
*   **Missed Branches (Ramificações):** Trata-se das lógicas condicionais (`if`, `else`, `switch`). Se você só testar o caminho feliz de um "if", a ramificação do caminho triste será marcada como falha (missed branch).
*   **Cyclomatic Complexity (Complexidade Ciclomática):** Indica quão confuso/complexo é um método. Cada `if`, `for` ou `switch` aumenta a complexidade.

**Interpretando as cores das Condicionais no JaCoCo:**
Ao abrir uma classe no relatório gerado, você verá losangos:
*   🟢 **Losango Verde:** Todas as ramificações de uma condicional (`true` e `false`) foram devidamente testadas.
*   🟡 **Losango Amarelo:** O bloco condicional está parcialmente testado (você testou o caminho de sucesso, mas esqueceu de escrever um teste forçando a falha, por exemplo).
*   🔴 **Losango Vermelho:** Toda a lógica desta ramificação foi ignorada na suíte de testes.
