# Capítulo 26: Lambda Expressions and the Streams API

A partir do Java 8, a linguagem sofreu uma de suas maiores revoluções ao abraçar o **Paradigma Funcional**. Antes, todo e qualquer código executável no Java tinha que pertencer obrigatoriamente a uma classe. As funções passaram a poder existir de forma (quase) independente.

## 1. O que são Expressões Lambda?

Lambdas são blocos de código (funções anônimas) que você pode passar como variáveis ou enviar por parâmetros para outros métodos.
Eles surgiram para encurtar absurdamente as antigas "Classes Anônimas".

**Sintaxe Básica:**
`(parâmetros) -> { corpo_da_função }`

Exemplos de conversão:
```java
// ANTES (Java 7 - Classe Anônima)
Calculador calc = new Calculador() {
    public int somar(int a, int b) {
        return a + b;
    }
};

// DEPOIS (Java 8 - Lambda)
Calculador calc = (a, b) -> a + b; 
// *Omissão de "return" e "{}" permitida em linha única.
```

## 2. Functional Interfaces (Interfaces Funcionais)

Para o Java conseguir ler um lambda (já que o Java ainda é puramente tipado e exige classes/interfaces), a Lambda Expression precisa ter como "tipo" uma **Interface Funcional**.

*   **O que é:** Uma interface que possui **APENAS UM** método abstrato.
*   **Anotação:** É boa prática usar `@FunctionalInterface` no topo da interface para o compilador travar se alguém tentar adicionar um segundo método nela por engano.

### Built-in Functional Interfaces (Interfaces embutidas no Java)
Você não precisa criar uma Interface Funcional toda vez que for criar uma Lambda. O pacote `java.util.function` traz várias prontas:
*   **`Predicate<T>`**: Recebe um argumento (T) e retorna um `boolean` (muito usado para condições/filtros através de um método interno chamado `.test(T)`).
*   **`Function<T, R>`**: Recebe um argumento de um tipo (T) e retorna um resultado de outro tipo (R) através do método `.apply(T)`.

---

## 3. A API de Streams (Manipulação em Massa de Dados)

O real motivo das lambdas terem sido criadas foi para facilitar a manipulação de Collections (Listas, Arquivos, Arrays) através de **Streams**.
Um Stream é um duto ou "pipeline" que percorre 3 etapas: **Fonte > Operações Intermediárias > Operação Terminal**.

### A. Fonte de Dados (Source)
De onde os dados vêm.
```java
// Exemplos de criação de um Stream:
IntStream.range(1, 10);
Stream.of("Apple", "Banana", "Cherry");
Arrays.stream(meuArray);
minhaList.stream();
Files.lines(caminhoDoArquivo); // Lendo arquivos de texto direto como Stream!
```

### B. Operações Intermediárias (Intermediate Operations)
Modificam, transformam ou filtram o Stream. Elas sempre retornam um *Stream atualizado*, permitindo o "encadeamento" de vários métodos.
*   `.filter(condicao)`: Deixa passar apenas os elementos que retornam true na condição (espera um `Predicate`).
*   `.map(transformacao)`: Converte o dado de um formato para outro (Ex: String para o tamanho dela em Integer). Espera uma `Function`.
*   `.sorted()`: Ordena a lista.
*   `.skip(n)`: Pula os `n` primeiros elementos.

*Dica de performance: Sempre faça os `.filter()` antes do `.sorted()`, pois ordenar listas gigantes e depois filtrá-las consome muito processamento à toa.*

### C. Operações Terminais (Terminal Operations)
Encerram o fluxo. Depois de uma operação terminal, o Stream acaba e devolve um resultado concreto.
*   `.forEach(acao)`: Executa um bloco de código para cada elemento final.
*   `.collect(Collectors.toList())`: Pega os itens restantes e agrupa de volta numa variável `List`.
*   `.sum()`, `.count()`, `.average()`: Reduz o stream a um único valor numérico.
*   `.findFirst()`: Pega o primeiro que sobrar (Costuma retornar um tipo `Optional`, então chamamos `.ifPresent()` para não estourar NullPointerException).

### Exemplo Completo de Stream:
```java
List<String> palavras = Arrays.asList("carro", "computador", "casa", "bola");

// Filtrar palavras que começam com 'c', convertê-las para maiúsculo e botar numa nova lista
List<String> palavrasComC = palavras.stream()
    .filter(p -> p.startsWith("c")) // Intermediária (Lambda retornando boolean)
    .map(p -> p.toUpperCase())      // Intermediária (Lambda retornando a String alterada)
    .sorted()                       // Intermediária
    .collect(Collectors.toList());  // Terminal
```
