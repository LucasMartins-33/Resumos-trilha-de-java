# Capítulo 12: Practical Project - Car Dealership

Neste capítulo prático, todo o conhecimento de Orientação a Objetos foi posto à prova simulando um sistema de Concessionária de Carros. O foco não foi apenas criar código, mas pensar como um engenheiro de software orientando a arquitetura do programa.

## 1. Como projetar um Sistema Orientado a Objetos (OO)

A dica de ouro ao começar a desenhar um sistema: **Pense em Substantivos (Nouns).** 
Um substantivo é uma pessoa, lugar ou coisa. Em um cenário de "venda de carros em uma concessionária", temos:
*   Dealership (A concessionária onde a aplicação vai rodar o `main`)
*   Employee (Funcionário)
*   Customer (Cliente)
*   Vehicle (Veículo)

Cada um desses substantivos torna-se uma **Classe** independente no seu projeto que irão interagir entre si através de instâncias (objetos).

## 2. Encapsulamento, Getters e Setters

É considerada uma **péssima prática** permitir que usuários (ou outras classes) modifiquem os atributos de um objeto diretamente (ex: `cliente.dinheiro = 50;`).
Para resolver isso, o Java usa o **Encapsulamento**:
1.  Você torna todas as variáveis da classe como `private`.
2.  Você cria métodos públicos para acessar (`get`) ou alterar (`set`) esses valores.

**Por que fazer isso?**
Porque nos métodos `set` você pode adicionar lógica de negócios. Por exemplo, se a concessionária quiser dar R$ 500 de bônus para todo cliente novo, você adiciona isso dentro do método `setCashOnHand()`, garantindo que essa regra seja sempre seguida de forma automática.

**Sintaxe de Encapsulamento:**
```java
public class Customer {
    // A variável é privada, não pode ser acessada por fora
    private double cashOnHand;
    private String name;

    // Método Getter: Para ler o valor
    public double getCashOnHand() {
        return cashOnHand;
    }

    // Método Setter: Para alterar o valor, com lógica embutida
    public void setCashOnHand(double amount) {
        this.cashOnHand = amount + 500; // Bônus de 500 embutido
    }
}
```
> **Dica de IDE:** No Eclipse, você pode gerar Getters e Setters automaticamente com: `Right Click > Source > Generate Getters and Setters`.

## 3. A Superclasse "Mãe de Todas": A classe `Object`

No Java, **todas as classes criadas herdam, por padrão, da classe `Object`**. Você não precisa escrever `extends Object`, isso acontece magicamente.
Isso significa que todo objeto que você criar já nasce com métodos poderosos embutidos, dos quais dois são extremamente importantes de serem **sobrescritos (Overridden)**:

### 3.1. Sobrescrevendo o método `.toString()`
Se você tentar dar um `System.out.println(meuCarro);`, o Java usará o `.toString()` padrão da classe `Object`, que imprime uma "sujeira" ilegível contendo o endereço de memória (ex: `Vehicle@7a81197d`). 

Para imprimir os dados formatados perfeitamente para humanos lerem, você deve **sobrescrever** esse método dentro da sua classe `Vehicle`:

```java
public class Vehicle {
    private String make;
    private String model;
    private double price;

    // Sobrescrevendo o comportamento padrão para uma string bonita
    @Override
    public String toString() {
        return "Vehicle [make=" + make + ", model=" + model + ", price=" + price + "]";
    }
}
```

### 3.2. Sobrescrevendo o método `.equals()`
Você criou duas instâncias diferentes de carro: `carroA` (Honda Accord, 10 mil) e `carroB` (Honda Accord, 10 mil). Ambos têm os exatos mesmos dados, mas moram em locais diferentes da memória Heap.
Se você comparar os dois com `carroA == carroB`, vai dar **falso** (pois o `==` compara o endereço da memória, não os atributos do carro).

Você deve sobrescrever o método `.equals()` para ensinar ao Java como comparar dois objetos da sua classe com base nos atributos internos deles.

```java
// O Eclipse também gera isso automaticamente (Source > Generate hashCode() and equals())
@Override
public boolean equals(Object obj) {
    // 1. Checa se as variáveis apontam pra exata mesma memória (mesmo objeto)
    if (this == obj) return true; 
    
    // 2. Tenta converter a genérica para um tipo Veículo para checar atributos
    Vehicle other = (Vehicle) obj; 
    
    // 3. Compara o chassi, modelo e preço ao invés do endereço.
    if (make.equals(other.make) && model.equals(other.model) && price == other.price) {
        return true;
    }
    return false;
}
```

## 4. Passando a própria classe usando `this` como argumento

Quando a classe `Customer` (Cliente) for comprar o carro, ele pede para o funcionário (`Employee`) processar os dados. No meio disso, o funcionário precisará dos dados do cliente. O cliente pode se passar como argumento num método dizendo "Tome, sou eu, pegue meus dados":

```java
public class Customer {
    public void purchaseCar(Vehicle v, Employee emp, boolean finance) {
        // O cliente invoca a ação do funcionário e passa 'this' (a si mesmo) como argumento
        emp.handleCustomer(this, finance, v);
    }
}
```
