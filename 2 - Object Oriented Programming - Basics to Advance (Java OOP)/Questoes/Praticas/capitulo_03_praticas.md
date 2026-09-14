📘 Capítulo 3: Pilares da Orientação a Objetos

**O Cenário:**
Você trabalha num e-commerce e tem um sistema rudimentar de produtos.

```java
public class Produto {
    private String nome;
    private double preco;
    
    // getters e setters
}
```

Sua missão é expandir esse sistema aplicando Herança, Polimorfismo e Abstração:

🟢 Atividade 3.1: Herança Básica
1. Crie uma classe `Livro` que estende `Produto`.
2. Adicione o atributo privado `String autor` e seus getters/setters.
3. Instancie um `Livro` no seu método `main`.

🟢 Atividade 3.2: Construtores na Herança
1. Adicione um construtor com `nome` e `preco` na classe `Produto`.
2. Na classe `Livro`, crie um construtor que receba `nome`, `preco` e `autor`.
3. Use a palavra-chave `super` para chamar o construtor da classe pai.

🟡 Atividade 3.3: Sobrescrita de Métodos (Overriding)
1. Crie um método `public void exibirDetalhes()` em `Produto` que imprima nome e preço.
2. Sobrescreva (`@Override`) o método `exibirDetalhes()` em `Livro` para imprimir também o autor.

🟡 Atividade 3.4: Polimorfismo em Ação (Upcasting)
1. No `main`, crie uma lista genérica: `List<Produto> carrinho = new ArrayList<>();`
2. Adicione na lista um objeto `Produto` e um objeto `Livro` recém instanciado.
3. Faça um laço `for` iterando sobre a lista chamando `exibirDetalhes()`. Observe o polimorfismo dinâmico.

🟡 Atividade 3.5: Downcasting Seguro
1. No mesmo laço `for` da atividade anterior, use o operador `instanceof`.
2. Se o produto for um `Livro`, faça o downcasting: `Livro l = (Livro) produto;`.
3. Imprima apenas o autor desse livro (usando `getAutor()`).

🟠 Atividade 3.6: Classes Abstratas
Descobrimos que não faz sentido vender um "Produto" genérico, ele deve ser de um tipo específico.
1. Altere a classe `Produto` adicionando a palavra `abstract`.
2. Tente instanciar um `new Produto()` e observe o erro.

🟠 Atividade 3.7: Métodos Abstratos
1. Adicione o método `public abstract double calcularFrete();` na classe `Produto`.
2. Implemente a regra de frete na classe `Livro` (ex: Livros têm frete fixo de R$ 5,00).

🔴 Atividade 3.8: Interfaces
O sistema agora lida com itens que podem ser baixados digitalmente.
1. Crie a interface `Digitalizavel` com o método `String gerarLinkDownload();`.
2. Crie uma classe `Ebook` que herda de `Livro` e implementa `Digitalizavel`.

🔴 Atividade 3.9: Múltiplas Interfaces (Contornando o Diamond Problem)
1. Crie a interface `Rentavel` com o método `double calcularAluguelMensal();`.
2. Faça o `Ebook` implementar também a interface `Rentavel`.
3. Implemente os dois métodos obrigatórios no `Ebook`.

🔴 Atividade 3.10: Composição ao invés de Herança
Em vez de herdar características, vamos usar Composição.
1. Crie uma classe `Fornecedor` com atributos `nomeEmpresa` e `cnpj`.
2. Na classe abstrata `Produto`, em vez de estender um `Fornecedor`, adicione um atributo `private Fornecedor fornecedor;`.
3. Ajuste os construtores para que todo produto dependa da composição de um fornecedor.
