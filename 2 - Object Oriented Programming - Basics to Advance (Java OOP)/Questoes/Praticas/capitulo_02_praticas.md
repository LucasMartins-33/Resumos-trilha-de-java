📘 Capítulo 2: Fundamentos da Orientação a Objetos

**O Cenário:**
Você está desenvolvendo o núcleo de um sistema escolar. Você tem uma classe base `Aluno` rudimentar e um sistema de notas.

```java
public class Aluno {
    String nome;
    int idade;
    double nota;
}
```

Sua missão é refatorar e testar essa estrutura resolvendo as atividades abaixo:

🟢 Atividade 2.1: O Primeiro Objeto
Crie a classe principal e instancie seu primeiro aluno.
1. Crie a classe `EscolaApp` com o método `main`.
2. Instancie um objeto da classe `Aluno` e atribua valores básicos para `nome` e `idade`.
3. Imprima o nome do aluno no console.

🟢 Atividade 2.2: Criando Construtores
A classe `Aluno` não tem construtor explícito.
1. Na classe `Aluno`, crie um construtor que receba `nome` e `idade`.
2. Atualize o `EscolaApp` para usar esse novo construtor.

🟡 Atividade 2.3: O Uso do `this`
Para evitar sombreamento de variáveis (shadowing):
1. Modifique os parâmetros do construtor para terem o exato mesmo nome dos atributos (`nome` e `idade`).
2. Utilize a palavra-chave `this` para referenciar os atributos da classe e faça a atribuição correta.

🟡 Atividade 2.4: Adicionando Comportamento
Objetos não têm apenas estado, mas também comportamento.
1. Adicione um método `public void estudar()` na classe `Aluno`.
2. O método deve imprimir: `"[nome do aluno] está estudando no momento."`
3. Chame esse método a partir do seu `EscolaApp`.

🟡 Atividade 2.5: Sobrecarga de Construtor
As vezes matriculamos alunos sem saber a idade de imediato.
1. Crie um segundo construtor na classe `Aluno` que receba apenas o `nome`.
2. Defina uma idade padrão (ex: 18) dentro deste construtor.

🟠 Atividade 2.6: Encapsulamento Básico
Deixar atributos expostos é perigoso.
1. Altere a visibilidade de `nome`, `idade` e `nota` para `private`.
2. Tente rodar o `EscolaApp` e veja o erro de compilação.
3. Crie os métodos *Getters* e *Setters* para resolver o problema.

🟠 Atividade 2.7: Validação de Estado
Os *Setters* servem para proteger o estado do objeto.
1. Modifique o método `setNota(double nota)`.
2. Adicione um `if` para garantir que a nota só seja alterada se estiver entre `0.0` e `10.0`.
3. Lance uma `IllegalArgumentException` caso contrário.

🔴 Atividade 2.8: Referências na Memória
Entenda como referências funcionam no Java.
1. No `EscolaApp`, crie `Aluno a1 = new Aluno("Lucas");`
2. Faça `Aluno a2 = a1;`
3. Altere a nota de `a2`. Imprima a nota de `a1`. Escreva em um comentário o motivo de ambas terem mudado.

🔴 Atividade 2.9: Tipos Primitivos vs Referência
1. Crie uma variável `int notaOriginal = 5;`.
2. Passe para um método auxiliar estático `alterarNota(int n)` que muda `n` para `10`.
3. Imprima `notaOriginal` e verifique que ela não mudou, contrastando com o comportamento da atividade anterior.

🔴 Atividade 2.10: Garbage Collection (Conceitual)
1. No `EscolaApp`, atribua `a1 = null;` e `a2 = null;`.
2. Escreva um comentário no código explicando em que momento o objeto Aluno criado passa a ser elegível para o Garbage Collector limpar da memória.
