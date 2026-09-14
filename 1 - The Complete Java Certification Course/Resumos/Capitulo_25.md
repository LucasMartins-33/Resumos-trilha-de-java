# Capítulo 25: Java 9 Features and the Shell (JShell)

O Java 9 introduziu uma ferramenta extremamente aguardada e popular: o **JShell**. Trata-se de um ambiente interativo (REPL - Read-Eval-Print Loop) que permite experimentar e testar códigos Java rapidamente sem precisar criar classes, métodos `main` ou compilar arquivos.

## 1. Iniciando o JShell

Para utilizar o JShell, você precisa ter pelo menos o Java 9 instalado na sua máquina.
1. Abra o seu Terminal ou Command Prompt.
2. Digite o comando: `jshell`
3. O ambiente interativo será iniciado: `Welcome to JShell -- Version 9...`

## 2. Escrevendo Código no JShell

A grande vantagem do JShell é a redução da verbosidade. Você pode digitar trechos de código diretamente.

*   **Semicolons opcionais:** Em instruções de linha única, você não precisa colocar o `;` no final.
    *   Exemplo: `System.out.println("Hello JShell")` (Funciona direto, e o uso da tecla `TAB` também aciona o autocompletar do JShell).
*   **Declaração de variáveis e métodos na raiz:** Você pode simplesmente digitar `String nome = "Lucas"` e a variável `nome` passa a existir na sessão.
*   **Blocos requerem Semicolon:** Se você usar chaves para abrir blocos de instrução (como `for`, `if` ou definir um método inteiro com chaves), as regras de sintaxe originais do Java se mantêm e você precisará usar os `;` corretamente dentro do bloco.

## 3. Comandos Úteis (Slash Commands)

O JShell possui comandos internos que começam com `/` para gerenciar o ambiente:

*   `/help`: Abre o menu de ajuda mostrando todos os comandos disponíveis.
*   `/list`: Lista os códigos/instruções ativas que você digitou até agora.
*   `/drop [nome]`: Apaga uma variável ou método específico da sessão atual (Ex: `/drop nome`).
*   `/edit [nome]`: Abre um pequeno bloco de notas/editor do JShell (Edit Pad) para você modificar a lógica de um método longo de forma mais confortável. Após editar, clique em *Accept* e feche.
*   `/vars`, `/methods`, `/types`, `/imports`: Listam respectivamente as variáveis criadas, os métodos criados, os tipos instanciados e os pacotes importados na sessão atual.
*   `/history`: Diferente do `/list` (que só mostra o que ainda está ativo), o history lista absolutamente tudo que foi digitado (incluindo erros e itens já deletados com `/drop`).
*   `/exit`: Encerra a sessão do JShell e volta para o Terminal padrão.

*(Nota: Toda vez que você encerra o JShell e o reinicia, o ambiente vem zerado. Variáveis antigas são perdidas).*

## 4. Testando Bibliotecas Externas (.jar) no JShell

O JShell é incrível para testar bibliotecas (arquivos `.jar`) que você baixou na internet ou compilou do seu próprio sistema, sem precisar montar um projeto inteiro no Eclipse.

Para iniciar o JShell carregando um `.jar` externo no Classpath, use a flag `-c` ou `--class-path` (em versões mais recentes) ao rodar no terminal:

```bash
# Inicia o JShell já "injetando" uma biblioteca externa
jshell -c /caminho/absoluto/ate/o/arquivo.jar

# (No Java 9 mais recente ou 11+, a sintaxe correta atualizada costuma ser --class-path)
jshell --class-path /caminho/absoluto/ate/o/arquivo.jar
```

Dentro do JShell, basta importar os pacotes daquela biblioteca (ex: `import com.meupacote.utils.*`) e começar a instanciar os objetos e testar seus métodos imediatamente!
