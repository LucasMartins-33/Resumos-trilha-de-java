# Capítulo 18: Practical Project: Stock Market Data

Neste capítulo prático, o objetivo foi aplicar todo o conhecimento acumulado sobre File Processing (Leitura de Arquivos) e Collections Framework (Listas e Dicionários) para processar um arquivo CSV com dados reais da bolsa de valores (ações da Apple).

O exercício consistiu em preencher a lógica de dois métodos cruciais em um projeto pré-existente.

## 1. Lendo os dados do Arquivo CSV (File Processing)

O primeiro método (`readFileData`) precisava ler o arquivo de texto linha por linha usando as técnicas aprendidas no Capítulo 14, ignorar o cabeçalho, e salvar as linhas em uma Lista.

**Anotações Práticas:**
*   **Try-with-Resources:** Usar `try (BufferedReader br = new BufferedReader(new FileReader(filePath)))` garante o fechamento automático da conexão com o disco, sem necessidade do bloco `finally`.
*   **Avançando o cursor:** Fazer um `br.readLine()` antes do laço `while` ajuda a pular a primeira linha do arquivo (que normalmente contém apenas os títulos/cabeçalhos, e não os dados).

## 2. Processando e Populando Coleções Aninhadas (`List<HashMap>`)

O segundo método (`populateStockFileData`) é onde a mágica das Coleções acontece. O objetivo era pegar as linhas de texto brutas e transformar em uma `List<HashMap<String, Double>>`.
Ou seja: Uma Lista (que representa o arquivo inteiro), onde cada slot dessa lista é um Dicionário/HashMap (representando 1 linha/registro da tabela), sendo a Chave o nome da coluna (String) e o Valor o preço da ação (Double).

**Passo a passo lógico implementado:**
1.  **Split:** Quebrar a linha de texto usando `String[] values = line.split(",");`. Isso separa os valores divididos por vírgula no CSV e os coloca em um Array.
2.  **Conversão de Tipos (Parsing):** O arquivo lido é 100% texto (`String`). Como os preços das ações são números decimais, usamos os **Wrapper Classes** para converter:
    *   `Double dVal = Double.parseDouble(stringNumber);`
3.  **Montando o HashMap (A Linha):** Criar um `HashMap<String, Double>` local. Fazer um laço de repetição nas colunas, atrelando o título do Cabeçalho (ex: "Open") com o valor em si (ex: 154.32), usando o método `.put(chave, valor)`.
4.  **Adicionando à Lista (A Tabela):** Pegar o HashMap pronto e adicioná-lo à Lista final usando `.add()`.

## 3. Dica Sintática: O Operador Diamante (Diamond Operator `<>`)

Até o Java 6, declarar coleções exigia repetição de código. A partir do Java 7, você não precisa repetir os tipos Genéricos no construtor da classe se eles já foram definidos na declaração da variável. O Java infere automaticamente usando os sinais de menor e maior (conhecido como Diamond Operator `<>`).

**Antes (Desnecessariamente longo):**
```java
HashMap<String, Double> map = new HashMap<String, Double>();
```

**Agora (Mais limpo):**
```java
HashMap<String, Double> map = new HashMap<>();
```
