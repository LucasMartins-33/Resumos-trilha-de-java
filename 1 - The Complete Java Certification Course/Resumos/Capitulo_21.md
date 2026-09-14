# Capítulo 21: Practical Project: Data Analytics Processor

Neste capítulo prático, o desafio foi combinar os conhecimentos de **Leitura de Arquivos (File Processing)**, **Coleções** e **Generics com Wildcards (Upper Bounds)**. 

O objetivo do projeto era ler o já conhecido arquivo `table.csv` com dados de ações financeiras e passar esses dados por um Processador capaz de calcular dinamicamente o valor Máximo, Mínimo ou a Média (Mean) de uma coluna específica escolhida pelo usuário.

## 1. Arquitetura da Solução

O projeto fornecia um pacote com a classe abstrata mãe `Aggregator` e três filhas que a estendiam: `MaxAggregator`, `MinAggregator` e `MeanAggregator`.
Cada classe filha sobrescrevia o método `calculate()` para realizar a sua própria lógica matemática.

O desafio central era construir a classe `AggregatorProcessor`, que precisava ser flexível o suficiente para aceitar QUALQUER tipo de Agregador (Máximo, Mínimo ou Média).

## 2. A Mágica dos Generics: O Upper Bound (`<T extends Classe>`)

Para que o Processador aceitasse dinamicamente qualquer classe agregadora, mas ao mesmo tempo bloqueasse objetos intrusos (como `String` ou `Employee`), foi declarada a seguinte sintaxe no cabeçalho da classe:

```java
// A classe AggregatorProcessor aceita um tipo T, desde que T seja um Aggregator ou filho dele.
public class AggregatorProcessor<T extends Aggregator> {
    
    T aggregator; // Pode virar um MaxAggregator, MinAggregator, etc.
    String file;
    
    public AggregatorProcessor(T aggregator, String file) {
        this.aggregator = aggregator;
        this.file = file;
    }
    // ...
}
```
Isso é a aplicação perfeita do Upper Bound! O Compilador garante a segurança de que `T` sempre terá os métodos definidos na classe mãe `Aggregator` (como os métodos `.add()` e `.calculate()`).

## 3. Lógica de Processamento do CSV e Arrays (0-based)

O método principal `runAggregator(int colIdx)` faz o seguinte:
1. Reutiliza o `StockFileReader` de projetos passados para ler o arquivo e retornar um `List<String> lines`.
2. Itera por essas linhas fazendo o clássico `line.split(",")` para obter um Array das colunas `String[] numbers`.
3. Ajusta o índice da coluna escolhida pelo usuário. Como humanos contam a partir do 1 e Arrays no Java começam no **0**, fazemos `colIdx--` antes de ler os dados do Array.
4. Converte o texto da coluna específica em número com `Double.parseDouble(numbers[colIdx])` e injeta no agregador via `aggregator.add(valor)`.
5. Por fim, chama o método polimórfico `aggregator.calculate()` e retorna o valor final daquela coluna inteira.

## 4. O Resultado no Client App
Dessa forma, a chamada na aplicação fica extremamente limpa, delegando todo o trabalho pesado para os Generics:

```java
// Instancia o Agregador desejado
MaxAggregator agg = new MaxAggregator();

// Passa o Agregador para o Processador Genérico
AggregatorProcessor<MaxAggregator> processor = new AggregatorProcessor<>(agg, "table.csv");

// Roda o processamento, por exemplo, na Coluna 1
double maxVal = processor.runAggregator(1); 
System.out.println(maxVal); // Saída: 144.22899
```
