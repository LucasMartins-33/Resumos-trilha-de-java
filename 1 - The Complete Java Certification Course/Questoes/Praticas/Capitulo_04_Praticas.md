# Questões Práticas - Capítulo 04 (Variáveis, Fluxo de Controle e Loops)

🟢 Nível 1: Declaração e Tipos de Dados
Cenário: Você está construindo a base de dados em memória para um sistema de controle de funcionários.
Sua Tarefa: Escreva um método chamado `cadastrarFuncionario`:
* Declare uma variável para a idade (inteiro), outra para o salário (ponto flutuante duplo) e uma terceira para indicar se o funcionário está ativo ou não (verdadeiro/falso).
* Inicialize-as com valores coerentes.
* Utilize o `System.out.println` para imprimir uma frase formatada concatenando todas essas variáveis (ex: "Idade: 30, Salário: 2500.50, Ativo: true").

🟡 Nível 2: Lógica Condicional Básica
Cenário: O sistema precisa aprovar ou reprovar o bônus de final de ano baseado em metas.
Sua Tarefa: Escreva o método `avaliarBonus(int notaDesempenho)`:
* Crie uma instrução `if / else`.
* Se a `notaDesempenho` for maior ou igual a 7, imprima "Bônus Aprovado!".
* Caso contrário, imprima "Bônus Reprovado.".

🟠 Nível 3: Condicionais Compostas (Short-circuit)
Cenário: Agora as regras do bônus ficaram mais rígidas. O bônus só é aprovado se o funcionário teve bom desempenho E já possui mais de 1 ano de empresa.
Sua Tarefa: Escreva o método `avaliarBonusEstrito(int nota, int tempoEmpresa)`:
* Utilize uma estrutura `if / else if / else` com o operador `&&`.
* Se a nota for >= 7 E o tempo for >= 1, imprima "Bônus Integral".
* Se a nota for >= 7, mas o tempo for < 1, imprima "Bônus Parcial".
* Qualquer outro cenário, imprima "Sem bônus".

🔴 Nível 4: A Estrutura Switch Case
Cenário: Você está criando um menu interativo de terminal para o caixa de um restaurante.
Sua Tarefa: Escreva o método `processarOpcaoMenu(int opcao)`:
* Implemente um `switch` baseado na variável `opcao`.
* Caso seja `1`, imprima "Pedido de Hambúrguer adicionado".
* Caso seja `2`, imprima "Pedido de Pizza adicionado".
* Caso seja `3`, imprima "Pedido de Salada adicionado".
* Adicione a cláusula `default` avisando "Opção Inválida".
* Não se esqueça de usar `break` para que os casos não sejam encadeados indevidamente.

🟣 Nível 5: O Loop While Clássico
Cenário: Um sistema de monitoramento de temperatura precisa baixar os graus de um reator superaquecido até atingir a temperatura ideal.
Sua Tarefa: Escreva o método `esfriarReator(int temperaturaAtual)`:
* Utilize um loop `while`.
* A condição do loop deve ser que a temperaturaAtual seja maior que 80.
* A cada iteração do loop, subtraia 5 da temperatura e imprima "Esfriando... Temperatura atual: X".
* Ao final do loop (fora dele), imprima "Reator estabilizado.".

🟤 Nível 6: A Importância do Do-While
Cenário: Você está criando um jogo de terminal que joga um dado. O dado deve rolar ao menos uma vez, mesmo se o jogador tiver 0 vidas (foi atingido no último turno), dando-lhe a chance de ressurreição.
Sua Tarefa: Escreva um bloco lógico contendo um `do-while`:
* Crie uma variável inteira `jogadas = 0`.
* Dentro do bloco `do`, imprima "Rolando o dado..." e incremente a variável `jogadas`.
* A condição do `while` deve ser rodar enquanto `jogadas < 3`.
* Prove conceitualmente (através da saída do código) que o código executa ao menos uma vez antes de validar a expressão de parada.

🔵 Nível 7: O Poder do Loop For
Cenário: Você precisa gerar um relatório que exibe a tabuada do número 7, do multiplicador 1 até o 10.
Sua Tarefa: Escreva o código usando o loop `for`:
* Configure a inicialização, a verificação e a atualização da variável do for na mesma linha.
* Dentro do loop, calcule e imprima a tabuada no formato "7 x N = RESULTADO".

🟢 Nível 8: Loops Aninhados (Nested Loops)
Cenário: Você precisa gerar um padrão de matriz na tela para uma parede de tijolos texturizada em formato quadrado (5x5).
Sua Tarefa: Utilize dois loops `for` aninhados:
* O loop externo controlará as linhas (de 1 a 5).
* O loop interno controlará as colunas (de 1 a 5).
* Dentro do loop interno, utilize `System.out.print("[X]")` (sem quebrar a linha).
* Assim que o loop interno terminar, utilize `System.out.println()` para pular para a próxima linha antes de continuar o loop externo.

🟡 Nível 9: Dominando Break e Continue
Cenário: Você está analisando um fluxo constante de logs. O sistema processa números de 1 a 100. Se encontrar um número múltiplo de 5, ele deve ser ignorado silenciosamente e passar para o próximo. Porém, se o número passar de 80, o processo deve ser abortado imediatamente por segurança.
Sua Tarefa: Crie um loop `for` de 1 a 100:
* Crie um `if` para verificar se o número atual é divisível por 5 (use o operador módulo `%`). Se sim, use `continue`.
* Crie um `if` para verificar se o número é maior que 80. Se sim, imprima "Limite de segurança atingido. Abortando." e use `break`.
* Se não entrar em nenhuma condição anterior, apenas imprima "Processando log ID: " + número.

🟠 Nível 10: O Desafio de Integração (Mini-Game RPG)
Cenário: Junte tudo o que aprendeu sobre Variáveis, If/Else e Loops! Você criará o loop de batalha entre um Herói e um Monstro.
Sua Tarefa: Escreva a lógica que atenda os requisitos:
* Declare a vida do herói como `100` e a do monstro como `50`.
* Crie um loop `while` que roda desde que AMBOS estejam vivos (vida > 0).
* Dentro do loop, cada turno o herói causa `15` de dano e o monstro causa `10`. Subtraia esses valores das respectivas vidas.
* Utilize `If/Else` para evitar que a vida de alguém fique negativa (se ficar menor que zero, crave em zero).
* A cada turno imprima o status: "Turno concluído: Herói (V) x Monstro (V)".
* Quando o loop terminar, crie um `If/Else` para imprimir quem foi o vencedor da batalha ou se houve um empate heroico.
