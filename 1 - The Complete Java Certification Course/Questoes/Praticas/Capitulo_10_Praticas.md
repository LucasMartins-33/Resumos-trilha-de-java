# Questões Práticas - Capítulo 10 (Strings, Nested Loops and Debugging)

🟢 Nível 1: Índices e Letras Avulsas
Cenário: Você precisa desenvolver a função básica que exibe o monograma (a letra inicial) do nome do cliente em um avatar circular.
Sua Tarefa: 
* Dada a variável `String nomeCliente = "Amanda";`.
* Extraia e guarde a PRIMEIRA letra do nome usando um método da classe String (cuidado, queremos a letra crua).
* Extraia também a ÚLTIMA letra dinamicamente (sabendo que o nome poderia ter qualquer tamanho, use o método matemático). Imprima ambas.

🟡 Nível 2: O Fatiador de Textos (`substring`)
Cenário: Os códigos de produto do estoque seguem o padrão `"CAT-12345"`, onde os 3 primeiros caracteres são sempre a categoria, o traço fica no meio, e o resto é o serial.
Sua Tarefa:
* Dada a string `"ELE-9080"`.
* Extraia a parte que representa a categoria (use os índices adequados) e guarde numa String `categoria`.
* Extraia a parte numérica (o número) passando apenas um parâmetro no `.substring()` (o índice de onde ele deve começar e ir até o infinito).
* Imprima ambos em linhas separadas.

🟠 Nível 3: A Armadilha de Memória e o Debugging Mental
Cenário: O código de validação de cupons abaixo está com um "bug" gravíssimo e você foi chamado para investigar.
Código de Partida:
```java
public void validarCupom(String cupomDigitado) {
    String cupomCorreto = new String("PROMO20");
    
    if (cupomDigitado == cupomCorreto) {
        System.out.println("Desconto aplicado!");
    } else {
        System.out.println("Cupom Inválido!");
    }
}
```
Sua Tarefa:
* Quando o usuário passa exatamente a String `"PROMO20"`, o sistema imprime "Cupom Inválido!". Reescreva o trecho problemático utilizando o método ensinado neste capítulo para que a lógica de negócio passe a funcionar e ignorando se ele digitar "promo20" (em minúsculas).

🔴 Nível 4: A Busca do Index (`indexOf`)
Cenário: Um analisador de URLs precisa extrair apenas o domínio de um site.
Sua Tarefa:
* Dada a String `url = "https://www.meusite.com/sobre-nos";`.
* Utilize o `.indexOf()` para procurar pela string `".com"`.
* Se o método não encontrar, imprima "Extensão comercial não encontrada".
* Se encontrar, imprima a posição numérica em que a palavra começou na string.

🟣 Nível 5: O Loop Reverso (Invertendo a String)
Cenário: Em entrevistas de emprego para programadores, um dos testes mais clássicos de lógica base (sem atalhos da API) é inverter manualmente uma String.
Sua Tarefa:
* Dada a palavra `String misterio = "RACECAR";`.
* Crie uma string vazia `String invertida = "";`.
* Crie um loop `for` configurado matematicamente para começar do ÚLTIMO índice de "misterio" e ir decremetando até o 0.
* Dentro do loop, concatene a letra isolada da vez dentro da variável `invertida`.
* Ao final do loop, faça um `if` para checar se a palavra original e a invertida são a mesma coisa, para declará-la um "Palíndromo".

🟤 Nível 6: Varredura de Caracteres no Loop
Cenário: O sistema precisa contar quantas letras 'A' ou 'a' o usuário digitou no campo de comentários.
Sua Tarefa:
* Dada a String `comentario = "O amor a arte eh maravilhoso";`
* Crie um contador de vogais (int).
* Faça um `for` clássico (do 0 até menor que o tamanho da string).
* Pegue a letra atual com `charAt(i)`. Se a letra for 'a' minúsculo ou 'A' maiúsculo, some 1 no contador. Imprima o total.

🔵 Nível 7: O Relógio de Engrenagens (Nested Loops Iniciais)
Cenário: Você precisa entender o tempo dos loops aninhados para simular um placar onde os minutos vão de 0 a 2 e os segundos de 0 a 59.
Sua Tarefa:
* Crie um loop `for` externo chamado `minuto` (vai de 0 até 2).
* Dentro dele, crie um `for` interno chamado `segundo` (vai de 0 a 59).
* Dentro do segundo, imprima: `"Tempo: " + minuto + ":" + segundo`. Observe em qual velocidade e ordem cada variável muda.

🟢 Nível 8: Matriz Bidimensional e Tabuleiro
Cenário: Criando um tabuleiro de Xadrez em texto. O tabuleiro tem 8 linhas e 8 colunas.
Sua Tarefa:
* Use loops aninhados (for dentro de for).
* Para cada iteração do loop interno, imprima `" [ ] "` usando `System.out.print` (sem o `ln` para não pular linha).
* Assim que sair do loop de dentro, dê um `System.out.println();` seco, apenas para causar a quebra de linha visual para a próxima linha do tabuleiro de fora.

🟡 Nível 9: Quebrando o Nested Loop Cedo (Performance)
Cenário: Escaneando listas longas. Queremos saber se nas caixas existe uma caixa preta.
Código de Partida:
```java
for (int andar = 1; andar <= 10; andar++) {
    for (int sala = 1; sala <= 10; sala++) {
        // Encontramos o que procurávamos logo na sala 2 do andar 1.
        if (andar == 1 && sala == 2) {
            System.out.println("Caixa Preta Encontrada!");
            // INSTRUÇÃO FALTANDO AQUI
        }
        System.out.println("Varegando Andar " + andar + " Sala " + sala);
    }
}
```
Sua Tarefa:
* No código acima, mesmo se acharmos a caixa na sala 2, o loop continuará varrendo todas as outras 98 salas inutilmente gastando energia do sistema.
* Use um `break` na instrução que falta para interromper o loop da `sala` (o interno). 
*(Nota: Para parar ambos os loops ao mesmo tempo de forma robusta no Java, muitas vezes teríamos que usar "Labels", mas neste cenário de exemplo focar no break interno já basta para exercitar a redução de processamento inútil).*

🟠 Nível 10: Integração Final - Encontrando a Palavra Secreta
Cenário: Junte Strings com Loops Aninhados! Vamos procurar o nome de um espião numa sopa de letrinhas (um pouco avançado!).
Sua Tarefa:
* Imagine uma variável String gigante contendo uma frase secreta sem espaços: `String bancoDados = "xptoagentebondmarcosbr";`.
* E uma string `String procurado = "bond";`.
* Utilizando apenas lógicas manuais (e não o `.contains()` do Java), tente criar um loop que lê a String zona por zona até achar a palavra. 
*(Dica: Faça um loop do começo ao fim do `bancoDados`, vá fatiando partes usando o `.substring()` e as compare com o `procurado`).* Imprima "Achou!" quando a lógica estiver correta.
