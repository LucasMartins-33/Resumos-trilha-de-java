# Questões Práticas - Capítulo 14 (File Processing and Exception Handling)

🟢 Nível 1: O Leitor de Teclado (Scanner)
Cenário: Você quer perguntar a idade do usuário para um validador de acesso.
Sua Tarefa: 
* Instancie um `Scanner` conectando-o ao `System.in`.
* Faça um print de pergunta na tela.
* Utilize o método `nextInt()` do scanner para capturar a resposta numa variável. Imprima a variável.

🟡 Nível 2: O Caos do Mundo Real (InputMismatchException)
Cenário: Você executou o programa do Nível 1, mas em vez de digitar `18`, o usuário digitou acidentalmente `DEZOITO`. O sistema engasgou e deu erro fatal na tela!
Sua Tarefa:
* Envolva as suas linhas de captura do `Scanner` do nível anterior dentro de um bloco `try { }`.
* Crie o bloco `catch (Exception e) { }` na sequência.
* Dentro do catch, imprima uma mensagem educada: "Erro: Você deveria ter digitado apenas números!".

🟠 Nível 3: O Guardião dos Recursos (Finally)
Cenário: Independentemente do usuário digitar a idade corretamente ou travar o sistema com letras, precisamos fechar a "porta" (o Scanner).
Sua Tarefa:
* Logo após a chave final do `catch`, adicione o bloco `finally { }`.
* Dentro do finally, invoque o método `.close()` da variável do seu scanner para desconectá-lo da memória do sistema. (Simule as variáveis visíveis, não se esqueça do escopo).

🔴 Nível 4: A Modernidade Mágica (Try-with-Resources)
Cenário: Na versão Java 7+, nós não sujamos as mãos com blocos `finally` ao lidar com recursos fecháveis.
Código de Partida:
```java
// O Scanner tradicional:
Scanner teclado = new Scanner(System.in);
String nome = teclado.nextLine();
// faltando o finally com teclado.close()
```
Sua Tarefa:
* Reescreva esse trecho isolado. 
* Remova a declaração do `Scanner` e coloque a inicialização (`Scanner teclado = new Scanner(...)`) **dentro** dos parênteses redondos logo após a palavra `try`.
* Coloque a lógica da leitura dentro das chaves do try.
* Note mentalmente que não será mais necessário chamar `close()`.

🟣 Nível 5: O Proclamador de Erros (`throws`)
Cenário: Você vai delegar um problema.
Sua Tarefa:
* Crie a assinatura de um método: `public void lerDiscoRigido()`.
* O Java não gosta de ações no disco. Sem escrever corpo nenhum ainda, adicione `throws IOException` no final da assinatura do método para alertar quem chamar este método no futuro de que o perigo existe. 

🟤 Nível 6: Jogando a Bomba Lógica Ativamente (`throw new`)
Cenário: O Java não solta Exceções sozinho para regras criadas PELA SUA empresa (como CPFs falsos). Precisamos fabricar e atirar as próprias exceções à força!
Sua Tarefa:
* Crie o método `public void avaliarCPF(String cpf) throws Exception`.
* Dentro dele, faça um `if`. Se o comprimento do CPF for menor que 11, execute a seguinte instrução vital:
* `throw new Exception("CPF muito curto! Fraude detectada.");`

🔵 Nível 7: Múltiplos Catchs em Cadeia
Cenário: Um único bloco pode falhar por falta de arquivo ou porque um arquivo existente tem dados bizarros.
Sua Tarefa:
* Escreva um bloco `try { ... }` fictício.
* Crie uma corrente logo abaixo: o primeiro `catch` deve tentar segurar especificamente um `FileNotFoundException`.
* O segundo `catch` abaixo dele deve segurar um genérico `Exception` caso o erro provocado no try não seja de arquivo (Ex: de Matemática).

🟢 Nível 8: A Solução Preventiva para NullPointer
Cenário: Tratar `NullPointerException` com try/catch é visto como amadorismo.
Código de Partida:
```java
String textoSecreto = null;
try {
    System.out.println(textoSecreto.length());
} catch (NullPointerException e) {
    System.out.println("Vazio!");
}
```
Sua Tarefa:
* Exclua o "try" e o "catch".
* Resolva o mesmo problema usando Lógica Preventiva pura (um simples `if / else`) conferindo se o objeto `!= null`. É muito mais elegante e rápido para o processador.

🟡 Nível 9: Exceção Customizada
Cenário: Jogar erros "Exception" é muito genérico. Você quer um erro só seu.
Sua Tarefa:
* Crie uma nova classe avulsa chamada `SenhasIguaisException`.
* Para ela se tornar um erro oficial aos olhos do Java, faça-a herdar da mãe de todos os erros de compilação genéricos: `extends Exception`.
* Escreva apenas o construtor dela que receba uma mensagem String e passe via `super(mensagem)` para o pai dela resolver.

🟠 Nível 10: O Ecossistema (Reunindo Tudo)
Cenário: Seu chefe mandou fazer um pequeno bloco à prova de balas.
Sua Tarefa (Apenas o pseudo-código ou as chamadas de métodos e blocos):
* Declare um método `login()` que tenha as cláusulas `throws`.
* No `main`, faça a chamada do `login()` rodeada por um moderníssimo `Try-with-Resources` (para ler a senha via teclado).
* Crie múltiplos `catch`s para segurar as falhas que o login puder cuspir.
* O fluxo deve provar que você domina abertura, checagem e fechamento moderno!
