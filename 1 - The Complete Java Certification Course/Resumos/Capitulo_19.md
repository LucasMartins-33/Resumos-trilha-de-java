# Capítulo 19: Java Generics

Este capítulo mergulha no recurso introduzido no Java 5 que revolucionou a segurança da linguagem: **Generics** (Tipos Genéricos, representados pelos símbolos `< >`).

## 1. Por que Generics existem? (Type Safety)

Java é uma linguagem fortemente tipada. Antes do Java 5, as Coleções eram **Raw Types** (Tipos Crús). 
Se você criasse um `ArrayList` sem especificar o tipo, poderia colocar de tudo dentro (uma `String`, um `Integer`, um `Boolean`). O problema é que, ao tirar os dados de lá (para o Java, tudo voltava como um `Object` genérico), você seria obrigado a fazer *casting* manual e, se errasse o tipo, o programa "quebrava" durante a execução (Runtime Error / ClassCastException).

**Com Generics (`ArrayList<String>`)**, você avisa ao Compilador exatamente o que pode entrar na lista. Se você tentar colocar um Número ali, o próprio editor (Eclipse/IntelliJ) bloqueia seu código na mesma hora (Compile-time Error). 

*Flexibilidade sem abrir mão da Segurança de Tipos (Type Safety).*

## 2. Classes Genéricas

Podemos criar nossos próprios "moldes" que aceitam qualquer tipo de dados. Basta usar os "Type Parameters" na declaração da classe. A convenção dita usar letras maiúsculas únicas (ex: `T` para Type, `E` para Element, `K` para Key, `V` para Value).

```java
// O T e o U são variáveis de "Tipo" que serão substituídas na hora da criação
public class Container<T, U> {
    T item1;
    U item2;
    
    public Container(T item1, U item2) {
        this.item1 = item1;
        this.item2 = item2;
    }
}

// Usando o Container:
Container<Integer, String> pote1 = new Container<>(12, "Doze");
Container<Double, Double> pote2 = new Container<>(1.5, 2.5);
```

## 3. Métodos Genéricos

Você também pode criar métodos genéricos. O "pulo do gato" sintático é que você **precisa declarar o tipo genérico ANTES do tipo de retorno do método**.

```java
// <E> declara que o método usa um tipo genérico E.
// Retorna um Set<E>
// Recebe dois argumentos Set<E>
public static <E> Set<E> union(Set<E> set1, Set<E> set2) {
    Set<E> result = new HashSet<>(set1);
    result.addAll(set2);
    return result; // Combina ambos removendo os duplicados
}
```

## 4. Wildcards (Curingas - `?`) e Polimorfismo

Aqui está a maior pegadinha dos Generics no Java:
Um `Accountant` (Contador) **é um** `Employee` (Funcionário).
Porém, uma `List<Accountant>` **NÃO É UMA** `List<Employee>`.

Se você criar um método que pede uma `List<Employee>`, você não pode passar uma `List<Accountant>`. O Java barra para manter a Type Safety rigorosa. Para resolver isso e aplicar o Polimorfismo nas coleções, usamos os **Wildcards (`?`)**.

*   **`<?>` (Unbounded Wildcard):** Significa "Lista de Qualquer Coisa". É praticamente a mesma coisa que voltar ao passado e usar o Raw Type.
*   **`<? extends ClasseMae>` (Upper Bound Wildcard):** Aceita a Classe Mãe e **qualquer filho dela** (Subclasses).
    *   Ex: `List<? extends Employee>` aceita `List<Employee>`, `List<Accountant>`, `List<Manager>`.
*   **`<? super ClasseFilha>` (Lower Bound Wildcard):** Aceita a Classe Filha e **qualquer classe pai acima dela**.
    *   Ex: `List<? super Employee>` aceita apenas `List<Employee>` e `List<Object>`. Não aceita filhos como Accountant.

### Exemplo do Upper Bound na prática:
```java
// Este método aceita uma lista de Funcionários ou de qualquer classe Filha de Funcionário!
public static void makeEmployeeWork(List<? extends Employee> employees) {
    for (Employee emp : employees) {
        emp.work();
    }
}
```
**Nota sobre downcasting:** Se usar Wildcards, o Java enxerga todos na lista como o "Teto" (Upper Bound). No exemplo acima, todos os elementos são tratados como `Employee` e terão acesso apenas aos métodos da classe `Employee`. Para forçar o acesso aos métodos do `Accountant`, seria necessário fazer um "Downcast" perigoso `((Accountant) emp).work()`, o que é desencorajado.
