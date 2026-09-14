# Capítulo 16: The Collections Framework

Este capítulo aborda a evolução do armazenamento de dados no Java. Diferente de um Array comum (que tem tamanho fixo e é limitado), o Collections Framework provê estruturas de dados super dinâmicas, que crescem sozinhas, encolhem e fazem malabarismos com os nossos dados, divididas em 3 grandes grupos: **Listas, Sets (Conjuntos) e Maps (Dicionários)**.

## 1. Listas (`ArrayList` e `LinkedList`)

As listas são estruturas dinâmicas que permitem repetições e preservam a ordem de inserção. 
A partir do Java 5, passamos a usar **Generics** (`<Tipo>`) para garantir *Type Safety* (Segurança de Tipos), obrigando as listas a guardarem apenas uma família de dados, evitando surpresas na hora de recuperar. 

**Importante:** Generics só aceitam **Classes (Reference Types)**, então não podemos usar primitivos. Use `Integer`, `Double`, `Boolean` etc, ao invés de `int`, `double`, `boolean`.

*   **`ArrayList`:** Por baixo dos panos, é um array que quando chega no seu limite de tamanho (default é 10), cria um array com o dobro do tamanho e copia os itens pra lá. É excelente para **recuperar dados** (ex: `list.get(300)`).
*   **`LinkedList`:** Os dados ficam como vagões de trem (nós), em que um aponta pro próximo. É excelente para **manipular dados** (inserir e deletar com alta frequência), mas é ruim para recuperação (pra pegar o item 300, precisa passar por 299 itens antes).

```java
import java.util.ArrayList;
import java.util.LinkedList;
import java.util.List; // Super tipo de ambos

// Polimorfismo: Declarar a Interface (List) e instanciar a Classe
List<String> animais = new ArrayList<String>();
animais.add("Leão");
animais.add("Gato");

List<Integer> numeros = new LinkedList<Integer>();
numeros.add(100);

// For-each moderno (A partir do Java 5)
for (String animal : animais) {
    System.out.println(animal);
}
```

## 2. Sets - Os Conjuntos Anti-Clones (`HashSet` e `LinkedHashSet`)

O foco do `Set` é **garantir a unicidade**. Ele NÃO permite elementos duplicados.
*   **`HashSet`:** Não garante nenhuma ordem de exibição.
*   **`LinkedHashSet`:** Garante a ordem de inserção.
*   **`TreeSet`:** Garante a "Ordem Natural" dos elementos (Ordem alfabética para Strings, Ordem crescente para Números).

### A Regra de Ouro dos Sets (equals e hashCode)
Quando você coloca `String` ou `Integer` em um Set, ele magicamente não deixa repetir.
**Mas**, quando colocamos os **nossos próprios Objetos** (ex: um objeto `Animal`), o Java não sabe que dois cachorros com a mesma idade e mesmo nome são "iguais". Para o Java, se eles estão em endereços de memória diferentes, são objetos distintos. 
Para que o `Set` consiga bloquear duplicatas de nossos objetos, **SOMOS OBRIGADOS a dar Override nos métodos `equals()` e `hashCode()`** da nossa classe!

## 3. Collections Methods & `Comparable`

A classe utilitária `Collections` traz muitos super-poderes. Mas o mais legal é o `Collections.sort()`.

**Como ordernar meus próprios objetos?**
O `Collections.sort(minhaLista)` funciona perfeitamente para Strings, mas ele vai dar erro se você tentar ordenar uma lista de `Employee` (Funcionários). O Java não tem bola de cristal para saber se você quer ordenar os funcionários por nome, por idade ou por salário. 
Você precisa ensinar isso a ele implementando a interface `Comparable`.

```java
// Implementando Comparable na sua classe:
public class Employee implements Comparable<Employee> {
    int salary;
    
    // Contrato da interface Comparable (Retorna -1, 0, ou 1)
    @Override
    public int compareTo(Employee outro) {
        if (this.salary < outro.salary) {
            return -1; // Joga pra cima
        } else if (this.salary > outro.salary) {
            return 1; // Joga pra baixo
        }
        return 0; // Se forem iguais
    }
}
```
**Outros Métodos Úteis de Listas/Sets:**
*   `lista1.addAll(lista2)`: Junta listas.
*   `lista1.removeAll(lista2)`: Remove os itens da lista1 que existirem na lista2.
*   `lista1.retainAll(lista2)`: Apaga tudo da lista 1, *exceto* o que estiver na lista 2.
*   `lista1.clear()`: Esvazia a lista.

## 4. Maps - A Estrutura de Dicionário (`HashMap`, `LinkedHashMap`, `TreeMap`)

Enquanto List e Set lidam com valores isolados, o **Map (Dicionário) trabalha com pares: Chave-Valor (Key-Value)**. A chave aponta para um valor (como uma palavra no dicionário aponta para o seu significado).
*   **A Chave DEVE SER ÚNICA.** (Se você der um `put` numa chave que já existe, ele **sobrescreve/substitui** o valor antigo).

```java
import java.util.Map;
import java.util.HashMap;

// <TipoDaChave, TipoDoValor>
Map<String, String> dictionary = new HashMap<String, String>();

dictionary.put("Brave", "Ready to face and endure danger or pain.");
dictionary.put("Brilliant", "Exceptionally clever or talented.");

// Resgatando o valor baseado na Chave
System.out.println(dictionary.get("Brave")); 
```

### Como iterar/navegar por um Map (Dicionário)?

```java
// Método 1: Apenas sobre as chaves (.keySet())
for (String word : dictionary.keySet()) {
    System.out.println("Palavra: " + word);
}

// Método 2: Sobre a Chave E o Valor juntos (.entrySet())
for (Map.Entry<String, String> entry : dictionary.entrySet()) {
    System.out.println("Palavra: " + entry.getKey());
    System.out.println("Definição: " + entry.getValue());
}
```
