# Anotações de Estudo: Java Core - Capítulo 3 (OOP Core Principles)

> [!NOTE]
> Este documento é o seu guia sobre os pilares da Orientação a Objetos em Java. Aqui você encontrará anotações, regras importantes e sintaxes baseadas no código do seu curso (ex: `ObjectInitializationOrder`).

---

## 11. Inheritance (Herança)

Herança permite criar novas classes com base em classes existentes. Reaproveita o código e estabelece uma relação de "é um" (Ex: `Phone` *é um* `Product`).
- **Sintaxe**: Usamos a palavra-chave `extends`. Em Java, **não existe herança múltipla** de classes (uma classe herda de no máximo UMA outra classe).

### A palavra-chave `super`
Referencia a classe pai. Serve para duas coisas principais:
1. Chamar métodos ou acessar variáveis da classe pai (`super.listVariants()`).
2. **Chamar o construtor da classe pai** (`super(name);`).
   > [!IMPORTANT]
   > A chamada ao construtor pai (`super()`) deve ser **obrigatoriamente a primeira linha** dentro do construtor filho!

### Operador `instanceof` e Casting
O Java precisa ter certeza do tipo do objeto em tempo de execução para permitir chamar métodos específicos do filho.
```java
Product product = new Phone(); // Funciona, Phone é Product
// product.ring(); // Erro! O compilador só enxerga a "casca" de Product.

// Verificando e convertendo (Downcasting) com segurança:
if (product instanceof Phone) {
    Phone phone = (Phone) product; // Casting de referência
    phone.ring(); // Agora funciona!
}
```

### Ordem de Inicialização (Initialization Order)
O que acontece por baixo dos panos quando você dá `new Child()`?
O arquivo [ObjectInitializationOrder.java](file:///C:/Users/lucas/OneDrive/Documentos/Cursos/learnit_java_core-master/learnit_java_core-master/src/com/itbulls/learnit/javacore/oop/inheritance/ObjectInitializationOrder.java) do seu projeto ilustra perfeitamente. A ordem é:
1. **Blocos Estáticos do Pai** (Somente na 1ª vez que a classe é carregada)
2. **Blocos Estáticos do Filho** (Somente na 1ª vez)
3. Blocos de Inicialização Não-Estáticos do Pai
4. Construtor do Pai
5. Blocos de Inicialização Não-Estáticos do Filho
6. Construtor do Filho

---

## 12. Polymorphism (Polimorfismo) e a palavra `final`

Polimorfismo ("muitas formas") permite que objetos diferentes respondam à mesma chamada de método de maneiras diferentes.
- **Dynamic Binding**: A decisão de qual método será executado ocorre em **tempo de execução** (Runtime), baseado no objeto real que está na memória.
- **`@Override`**: Anotação (metadado) opcional, mas recomendada. Informa ao compilador: "Estou sobrescrevendo este método do pai". Se você errar a assinatura, o compilador avisa.

### Overriding vs Overloading
- **Overriding (Sobrescrita)**: Mesma assinatura, mesmo nome. Comportamento diferente na classe filha (Polimorfismo).
- **Overloading (Sobrecarga)**: Mesmo nome, mas **parâmetros diferentes** (assinaturas diferentes). Pode ocorrer na mesma classe.

### O uso do `final`
O modificador `final` muda de significado dependendo de onde é aplicado:
- **Classe `final`**: Não pode ser estendida/herdada (Ex: `String`).
- **Método `final`**: Não pode ser sobrescrito (`@Override`) pelas classes filhas.
- **Variável/Parâmetro `final`**: Seu valor não pode ser alterado depois de inicializado (funciona como uma constante local).

---

## 13. A palavra-chave `static`

Algo que é `static` pertence à **Classe** e não ao **Objeto**.
- **Static Variables**: Existe em uma única cópia. Se um objeto alterar o valor, todos os outros verão o novo valor. (Ex: um contador de instâncias).
- **Static Binding**: Como métodos estáticos não pertencem ao objeto, o Java decide qual método chamar em **tempo de compilação**, anulando o polimorfismo dinâmico.
- Por que `static` é mal visto em POO pura? Porque impede a sobrescrita (quebra o polimorfismo), cria estados globais difíceis de rastrear e dificulta testes unitários.

> [!TIP]
> Você pode usar um `import static java.util.Arrays.*;` para não precisar ficar escrevendo `Arrays.sort()`. Basta chamar `sort()` direto.

---

## 14. Encapsulation (Encapsulamento)

É o princípio de proteger os dados e ocultar a implementação interna, agrupando campos e métodos numa única unidade.

### Modificadores de Acesso (Do mais restritivo ao mais aberto)
1. **`private`**: Visível APENAS dentro da própria classe.
2. **`package-private` (default)**: Não se escreve nada. Visível APENAS dentro do mesmo pacote (package).
3. **`protected`**: Visível no mesmo pacote E em qualquer classe que herde dela (mesmo em pacotes diferentes).
4. **`public`**: Visível de qualquer lugar do projeto.

### Regra de Ouro da Sobrescrita (Overriding)
Você **nunca pode reduzir a visibilidade** de um método ao sobrescrevê-lo.
* Se no pai o método é `protected`, no filho ele pode ser `protected` ou `public`. Nunca `private`! (Gera erro de compilação).

---

## 15. A Classe `Object` e JNI

No Java, **toda classe herda implicitamente da classe `Object`**. É por isso que você pode chamar alguns métodos mesmo numa classe vazia.
- **JNI (Java Native Interface)**: Permite que código Java chame programas escritos em C/C++. Identificamos isso pelo modificador `native`. (Ex: `public final native Class<?> getClass();`).

### Principais métodos herdados de `Object`
- `getClass()`: Retorna o metadado da classe para uso com Reflection.
- `hashCode()`: Retorna um número inteiro, usado muito em coleções como `HashMap`.
- `equals(Object obj)`: Compara objetos. Por padrão, compara apenas o endereço de memória (referência). Devemos sobrescrevê-lo para comparar o estado lógico (ex: se as placas de dois carros são iguais, é o mesmo carro).
- `clone()`: Usado para copiar um objeto. Atenção: por padrão, faz cópia rasa (shallow copy), copiando referências, não os objetos internos.
- `toString()`: Converte o objeto para String. Sempre sobrescreva para facilitar o debug!
- `wait()`, `notify()`, `notifyAll()`: Métodos usados para Sincronização e Multithreading.
- `finalize()`: Era usado pelo Garbage Collector antes de destruir o objeto. **Está obsoleto (`@Deprecated`)** e não deve mais ser usado, pois causa problemas de vazamento de memória.
