📘 Capítulo 7: Exercícios Críticos para Entrevistas

**O Cenário:**
Muitas entrevistas de Java testam o seu conhecimento profundo de pequenos detalhes da Orientação a Objetos.

Sua missão é programar pequenos "edge cases" em Java que costumam ser pegadinhas em entrevistas:

🟢 Atividade 7.1: Entendendo o Modificador `final` em Classes
1. Crie uma classe `SegurancaSistema` e coloque a palavra reservada `final` nela.
2. Tente criar uma classe `Hacker` que faz `extends SegurancaSistema`.
3. Valide que o compilador não permite a herança.

🟢 Atividade 7.2: Entendendo o Modificador `final` em Métodos
1. Crie uma superclasse com um método `public final void gerarLog()`.
2. Em uma subclasse, tente sobrescrever (`@Override`) o método `gerarLog()`.
3. Verifique o erro de compilação.

🟡 Atividade 7.3: Herança Oculta (A classe Object)
1. Crie uma classe simples `Carro` (sem atributos).
2. Instancie e chame o método `toString()`. De onde esse método veio se você não o programou?
3. Sobrescreva o método `toString()` para retornar `"Meu Carro Customizado"`.

🟡 Atividade 7.4: O Contrato `equals()`
1. Adicione os atributos `marca` e `modelo` na classe `Carro`.
2. Crie duas instâncias separadas com os mesmos dados (ex: "Fiat", "Uno").
3. Imprima `carro1.equals(carro2)`. Por que o resultado é falso por padrão?
4. Sobrescreva o `equals()` para comparar a string dos modelos.

🟠 Atividade 7.5: O Contrato `hashCode()`
Em entrevistas, se você sobrescreve o `equals`, a pergunta seguinte é sempre sobre o `hashCode`.
1. Insira seus carros em um `HashSet`.
2. Mesmo sendo iguais pela sua nova regra de `equals`, o Set pode permitir duplicatas se o `hashCode` for diferente.
3. Sobrescreva o `hashCode()` retornando `marca.hashCode()`.

🟠 Atividade 7.6: Method Hiding (Métodos Estáticos não são Polimórficos)
1. Crie a classe `Pai` com `public static void saudar() { print("Pai"); }`.
2. Crie a subclasse `Filho` com `public static void saudar() { print("Filho"); }`.
3. Faça o teste de pegadinha: `Pai p = new Filho(); p.saudar();`. Qual foi a saída? Comente o porquê.

🔴 Atividade 7.7: Coesão vs Acoplamento (Refatoração Mental)
1. Escreva uma classe chamada `SuperGestorDeVendas` que calcula impostos, envia emails, e salva no banco de dados.
2. No comentário do código, explique por que essa classe tem Baixa Coesão e Alto Acoplamento.

🔴 Atividade 7.8: Default Methods (Java 8+)
1. Crie uma interface `Relatorio` com um método abstrato `gerar()`.
2. Adicione um método com a palavra-chave `default`: `default void imprimir() { System.out.println("Imprimindo..."); }`.
3. Crie uma classe que implementa `Relatorio` e observe que não é necessário implementar o método `imprimir`.

🔴 Atividade 7.9: Construtores Privados
1. Crie uma classe `ConfiguracaoApp` e coloque seu construtor como `private`.
2. Como outras classes podem utilizá-la agora?
3. Crie um método estático `public static ConfiguracaoApp getInstance()` (O padrão Singleton inicial).

🔴 Atividade 7.10: Early Binding vs Late Binding
1. Crie um método sobrecarregado (Overload): `void ler(Livro l)` e `void ler(Ebook e)`.
2. Crie métodos sobrescritos (Override) numa hierarquia.
3. Demonstre em comentários que a resolução de Overload é Early Binding (compilação) e Override é Late Binding (runtime).
