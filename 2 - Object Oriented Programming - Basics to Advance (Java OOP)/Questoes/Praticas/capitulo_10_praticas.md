📘 Capítulo 10: Princípios GRASP na Prática

**O Cenário:**
Você está criando um sistema de Ponto de Venda (PDV) de uma loja. Tem as classes `Venda`, `ItemVenda` e `Produto`.

```java
public class Venda {
    // Lista de itens
}
public class ItemVenda {
    int quantidade;
    Produto produto;
}
public class Produto {
    double preco;
}
```

Sua missão é atribuir responsabilidades aos objetos corretos usando GRASP.

🟢 Atividade 10.1: Information Expert (Subtotal)
1. Precisamos calcular o subtotal de um `ItemVenda` (quantidade * preço).
2. Qual classe possui a informação necessária para fazer isso?
3. Crie o método `calcularSubtotal()` na classe correta e implemente-o.

🟢 Atividade 10.2: Information Expert (Total da Venda)
1. Precisamos calcular o Total da Venda somando todos os itens.
2. Qual classe tem a lista de itens e é a "Especialista"?
3. Crie o método `calcularTotal()` na classe `Venda` iterando pelos itens.

🟡 Atividade 10.3: Creator (Criador de Itens)
1. Precisamos adicionar um novo item à Venda. Quem deve instanciar o `new ItemVenda()`?
2. Seguindo o padrão Creator, a classe `Venda` compõe e agrega itens.
3. Crie o método `adicionarProduto(Produto p, int qtd)` na classe `Venda` que internamente cria o `ItemVenda`.

🟡 Atividade 10.4: Controller (A Interface do Usuário)
1. O caixa apertou o botão "Finalizar Venda" na tela (UI). A UI não deve chamar objetos de negócio diretamente.
2. Crie uma classe `CaixaController`.
3. Adicione o método `finalizarVenda(int idCaixa)` para ser o ponto de entrada da operação.

🟠 Atividade 10.5: Low Coupling (Baixo Acoplamento)
1. Suponha que o Controller precise avisar o Estoque que os produtos saíram.
2. Se o `CaixaController` chamar o BD de Estoque direto, o acoplamento sobe.
3. Crie uma interface `ServicoEstoque` e faça o Controller depender apenas da interface.

🟠 Atividade 10.6: High Cohesion (Alta Coesão)
1. O `CaixaController` não deve formatar o recibo (string complexa) para a impressora, pois perde coesão.
2. Crie a classe `FormatadorRecibo`.
3. Mova a responsabilidade de gerar o texto do recibo para ela.

🔴 Atividade 10.7: Polymorphism (Polimorfismo para Regras)
1. A loja aceita pagamentos em `Dinheiro`, `CartaoCredito` e `Pix`.
2. Em vez de fazer um `switch` enorme na `Venda` para processar pagamento, aplique polimorfismo.
3. Crie a interface `MetodoPagamento` com o método `processar(double valor)`.

🔴 Atividade 10.8: Pure Fabrication (Fabricação Pura)
1. Precisamos salvar a `Venda` no banco. Nem a Venda nem o Controller devem ter `sql = "INSERT INTO..."`.
2. O GRASP sugere inventar uma classe que não existe no mundo real.
3. Crie o `VendaRepository` (Fabricação Pura) para lidar com o SQL.

🔴 Atividade 10.9: Indirection (Indireção)
1. O sistema agora envia Nota Fiscal via API de terceiros (SEFAZ).
2. Não ligue a Venda diretamente à biblioteca externa (alto acoplamento).
3. Crie uma classe intermediária (Indireção) `AdaptadorSEFAZ` que mascara a comunicação.

🔴 Atividade 10.10: Protected Variations (Variações Protegidas)
1. As APIs de pagamento mudam constantemente de versão.
2. Garanta que o seu core de negócio não quebre usando injeção de dependência via interfaces para todas as integrações externas vistas acima, protegendo-se das variações.
