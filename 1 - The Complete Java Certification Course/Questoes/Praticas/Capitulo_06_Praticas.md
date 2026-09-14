# Questões Práticas - Capítulo 06 (Understanding Methods)

🟢 Nível 1: Entendendo o Retorno (Void vs Tipos Primitivos)
Cenário: Você tem uma classe chamada `Calculadora` que possui um método que imprime uma soma, mas precisamos de um método que de fato *retorne* o cálculo para o sistema.
Sua Tarefa: 
* Crie um método `public static int somar(int a, int b)`.
* Dentro do método, armazene a soma numa variável e retorne-a usando a instrução `return`.

🟡 Nível 2: Chamada de Método Estático
Cenário: Com o seu método criado no nível 1, agora ele deve ser testado.
Sua Tarefa:
* Dentro do método `public static void main(String[] args)` da sua aplicação, chame o método `somar` passando os números 10 e 25.
* Armazene o retorno em uma variável `resultado`.
* Imprima a variável na tela usando `System.out.println()`.

🟠 Nível 3: Refatoração: Removendo o Código Repetido
Cenário: O método `main` está poluído com lógicas repetitivas para exibir o cabeçalho do sistema para o usuário toda vez.
```java
public static void main(String[] args) {
    System.out.println("==================");
    System.out.println(" BEM-VINDO AO APP ");
    System.out.println("==================");
    // Algum código...
    System.out.println("==================");
    System.out.println(" BEM-VINDO AO APP ");
    System.out.println("==================");
}
```
Sua Tarefa:
* Extraia esses 3 `println` repetidos para um novo método chamado `imprimirCabecalho()`.
* Garanta que ele não retorne nada (`void`).
* Substitua os códigos repetidos no `main` pelas chamadas desse novo método.

🔴 Nível 4: A Magia do `new` (Iniciando na Orientação a Objetos)
Cenário: A sua equipe decidiu que a classe `Calculadora` não deveria mais ter métodos utilitários estáticos, para permitir que no futuro cada calculadora guarde o histórico de contas.
Sua Tarefa:
* Remova a palavra-chave `static` do método `somar()` do Nível 1.
* Ao tentar chamá-lo no `main` (`Calculadora.somar()`), o compilador dará erro. Conserte o `main`! Você precisará instanciar um objeto usando `Calculadora minhaCalc = new Calculadora();` e invocar a soma a partir desse objeto.

🟣 Nível 5: O Argumento do Main (String[] args)
Cenário: Você está criando um pequeno utilitário de terminal. Quando rodar no terminal (`java App Lucas`), o app deve lhe dar as boas vindas.
Sua Tarefa:
* Dentro do método `main`, acesse a posição zero do array de argumentos passado pela JVM: `args[0]`.
* Armazene isso numa variável String.
* Imprima "Olá, " + variavel.

🟤 Nível 6: Encadeamento e Retorno como Argumento
Cenário: Você precisa multiplicar o resultado de duas somas diferentes, e quer fazer isso num código super curto.
Sua Tarefa:
* Supondo que você devolveu o `static` pro método `somar()`, e também criou um `public static int multiplicar(int a, int b)`.
* No método `main`, execute uma multiplicação usando a sintaxe onde as somas são passadas DIRETAMENTE dentro da chamada: `multiplicar(somar(2,3), somar(4,1))`.

🔵 Nível 7: Modificadores de Acesso e Segurança
Cenário: Você criou uma classe `DatabaseConnector` com o método `conectar()`. O problema é que o método `gerarSenhaDoBanco()` está exposto para o público.
```java
public class DatabaseConnector {
    public static void conectar() {
        String senha = gerarSenhaDoBanco();
        System.out.println("Conectando com " + senha);
    }
    public static String gerarSenhaDoBanco() {
        return "123456"; 
    }
}
```
Sua Tarefa:
* Troque o modificador do método `gerarSenhaDoBanco` para que *apenas* a classe `DatabaseConnector` consiga visualizá-lo. (Dica: substitua `public`).

🟢 Nível 8: Tipos de Dados Mistos nos Parâmetros
Cenário: O sistema precisa de um método genérico para gerar crachás de funcionários formatados.
Sua Tarefa:
* Crie o método `public static String formatarCracha(String nome, int idade, double salario)`.
* Dentro do corpo do método, retorne (NÃO imprima, use `return`) uma única String concatenando os 3 argumentos, como por exemplo: "Nome: Lucas | Idade: 30 | Salário: 2500.5".

🟡 Nível 9: Package-Private e Omissão
Cenário: O arquiteto do software revisou seu código e disse que o método `formatarCracha` não deveria ter a palavra `public`.
Sua Tarefa:
* Remova o `public`.
* O que acontece com a capacidade de outras classes chamarem esse método? (Apenas responda mentalmente ou coloque como comentário em cima do método para indicar que agora ele é de visibilidade "default" e preso àquele pacote).

🟠 Nível 10: Integração Final (O Serviço Bancário)
Cenário: Juntando todos os aprendizados sobre chamadas, retornos, instanciações e parâmetros, você criará a ponte de um serviço bancário em miniatura.
Sua Tarefa:
* Crie uma classe `BancoUtils`. Nela, crie os métodos ESTÁTICOS `public static double checarTaxaDeSaque()` (que retorna 2.50) e `public static boolean isDomingo()` (retorna false).
* Crie a classe `ContaCorrente`. Crie o método NÃO-ESTÁTICO `public double sacar(double valorDesejado)`.
* A lógica do `sacar` (dentro da ContaCorrente): adicione a `valorDesejado` a taxa pega pelo `BancoUtils.checarTaxaDeSaque()`. Subtraia o saldo (assuma um saldo imaginário ou declare-o na classe) e retorne o saldo atual.
* No `main` (Classe Application): Instancie a `ContaCorrente` com o `new`. Chame o `.sacar(100.0)`. Imprima o retorno final para garantir que deu certo!
