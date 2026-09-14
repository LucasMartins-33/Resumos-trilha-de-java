# Questões Teóricas - Capítulo 25 (Java 9 Features and the Shell)

**1. A Chegada do Shell:** O que é o JShell e a partir de qual versão do Java ele foi introduzido oficialmente no ecossistema da linguagem?
> [!faq]- 👀 Ver Resposta
> O JShell é um ambiente de linha de comando interativo do Java. Ele foi introduzido oficialmente no **Java 9** (JDK 9) para facilitar a prototipagem rápida e o ensino.

**2. O Acrônimo Famoso (REPL):** Ferramentas iterativas de terminal como o JShell ou o terminal do Python são conhecidas na computação pela sigla REPL. O que essa sigla significa e como ela traduz o funcionamento da ferramenta?
> [!faq]- 👀 Ver Resposta
> REPL significa **R**ead, **E**valuate, **P**rint, **L**oop. O terminal Lê a linha digitada, Avalia/processa o código no mesmo instante, Imprime o resultado final imediatamente na tela, e entra em Loop esperando o seu próximo comando, como um fluxo contínuo sem telas de carregamento.

**3. Vantagem Competitiva contra IDE:** Qual é a principal vantagem e utilidade do JShell no dia-a-dia em relação a construir códigos em uma IDE tradicional como o Eclipse ou IntelliJ?
> [!faq]- 👀 Ver Resposta
> O JShell elimina o "peso/boilerplate". Na IDE, para você testar rapidamente qual é o resultado de "Math.pow(2, 5)", você precisaria criar um Projeto inteiro, criar um arquivo `.java`, escrever a "class", o método "main", rodar o build e compilar. No JShell, você abre o terminal, digita "Math.pow(2,5)", aperta Enter e a resposta sai em 1 segundo.

**4. O Disparo Inicial:** Sabendo que o Java JDK já está instalado na máquina, como abrimos e ativamos a ferramenta JShell usando apenas o CMD do Windows ou Terminal do Linux/Mac?
> [!faq]- 👀 Ver Resposta
> Basta abrir a janela do CMD normal do sistema, digitar simplesmente `jshell` e apertar Enter. O prompt de comando mudará o texto indicativo para `jshell>` provando que você está imerso no Java interativo.

**5. Quebrando as Regras do Java:** No JShell, você precisa obrigatoriamente criar e empacotar seu código na famosa casca `public class X` e ter um `public static void main(String[] args)` para conseguir imprimir variáveis soltas?
> [!faq]- 👀 Ver Resposta
> Não, e essa é a maior mágica dele! Ele é um ambiente limpo. Você pode digitar apenas `System.out.println("Oi");` diretamente ou simplesmente digitar `10 + 5` que o ambiente aceitará e cuspirá a saída sem a obrigatoriedade de construir classes ou métodos Main.

**6. As Variáveis Invisíveis (Scratch Variables):** Se você digitar apenas `23 * 2` e apertar Enter no JShell (sem criar uma variável `int x = ...`), como a plataforma guarda e rotula o resultado dessa conta temporariamente para você reutilizá-lo depois?
> [!faq]- 👀 Ver Resposta
> O JShell cria automaticamente variáveis ocultas ou temporárias sequenciais, cujos nomes geralmente começam com um cifrão (Ex: `$1`, `$2`, `$3`). Se o resultado deu 46 e ele gravou no `$1`, basta digitar `$1 + 4` em seguida e ele saberá que a resposta é 50.

**7. Auditando o Banco de Memória:** Se você passou uma hora testando dezenas de declarações de variáveis no JShell e perdeu a conta do que tem gravado nele, qual comando interno nativo lista tudo e devolve os valores vivos do cache no momento?
> [!faq]- 👀 Ver Resposta
> Devemos invocar os utilitários especiais (que começam com a barra `/`). No caso, usa-se o comando `/vars` (ou `/v`).

**8. Métodos e Histórico:** Semelhante ao comando do cache de variáveis, quais comandos utilizamos para consultar todos os métodos customizados que criamos na sessão, e o histórico total de scripts?
> [!faq]- 👀 Ver Resposta
> Usa-se `/methods` (ou `/m`) para consultar os escopos funcionais da sessão. E usa-se `/history` para ver absolutamente cada linha de código que você teclou no terminal desde que o JShell foi aberto.

**9. Complexidade Suportada (Blocos):** O JShell é limitado apenas a instruções minúsculas e de "linha única" (Single Line Statements) de soma e concatenação, ou é possível digitar estruturas imensas de chaves, `ifs`, e `loops` dentro dele?
> [!faq]- 👀 Ver Resposta
> Ele suporta totalmente lógicas e blocos densos (Multi-line). Você pode começar a declarar um loop `for (int i = 0; i < 5; i++) {`, apertar Enter, e o prompt mudará seu ícone (geralmente para um `...>`) indicando que ele "sabe" que as chaves não foram fechadas, esperando que você digite os conteúdos das linhas de dentro até dar o fechamento `}` e o Enter final para executar o bloco massivo inteiro na mesma hora.

**10. Fuga do Sistema:** Assim como um aplicativo, como nós desligamos ou fechamos educadamente o ambiente JShell pelo terminal quando não precisamos mais brincar nele, de modo a voltar para o prompt comum de arquivos do Windows?
> [!faq]- 👀 Ver Resposta
> A instrução final é simplesmente usar o comando `/exit`. Ele descarrega e limpa a memória daquela sessão, matando o REPL interativo e devolvendo você para o C:\ habitual do sistema.




