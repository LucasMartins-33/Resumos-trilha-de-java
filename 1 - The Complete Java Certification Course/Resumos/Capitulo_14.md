# Capítulo 14: File Processing and Exception Handling

Este capítulo cobre como o Java se comunica com o mundo exterior (lendo dados do teclado e de arquivos no HD) e como lidar elegantemente com os erros (Exceções) que frequentemente acontecem durante essas operações.

## 1. Lendo input do Teclado (Scanner)

Para criar aplicações interativas via terminal, o Java utiliza a classe `Scanner` atrelada à entrada padrão do sistema (`System.in`).

**Sintaxe Básica:**
```java
import java.util.Scanner;

public class Application {
    public static void main(String[] args) {
        // Inicializando o Scanner para ler o teclado
        Scanner input = new Scanner(System.in);
        
        System.out.println("Digite o seu nome:");
        
        // Pausa o programa e aguarda o usuário digitar algo e apertar ENTER
        String nome = input.nextLine(); 
        
        System.out.println("Olá, " + nome + "!");
        
        input.close(); // Sempre feche os recursos quando não precisar mais deles!
    }
}
```

## 2. Tratamento de Exceções (`try`, `catch`, `finally`)

Exceção é o termo chique para **Erro**. Como ler um arquivo no HD pode dar errado (o arquivo pode ter sido deletado, o HD pode estar corrompido), o Java nos OBRIGA a tratar essas possibilidades.

*   **`try`:** Tente rodar este bloco de código perigoso.
*   **`catch`:** Se o erro X acontecer, capture-o e rode esse bloco de emergência (em vez de travar o programa e cuspir um erro feio na tela).
*   **`finally`:** Bloco opcional que SEMPRE vai rodar, quer tenha dado erro ou não. Historicamente, muito usado para fechar arquivos/conexões à força.

### Anatomia de um Try/Catch/Finally (Estilo Antigo - Pré Java 7)
```java
import java.io.*;

public class App {
    public static void main(String[] args) {
        File file = new File("meuarquivo.txt");
        FileReader fileReader = null;
        BufferedReader bufferedReader = null;

        try {
            // Tenta abrir o arquivo (pode lançar FileNotFoundException)
            fileReader = new FileReader(file);
            bufferedReader = new BufferedReader(fileReader);

            String line = bufferedReader.readLine(); // Pode lançar IOException
            while (line != null) {
                System.out.println(line);
                line = bufferedReader.readLine();
            }
            
        } catch (FileNotFoundException e) {
            System.out.println("Ops! O arquivo não foi encontrado: " + file.getName());
        } catch (IOException e) {
            System.out.println("Problema na leitura do disco!");
        } finally {
            // O Finally sempre roda. Aqui fechamos as portas.
            try {
                if (bufferedReader != null) bufferedReader.close();
                if (fileReader != null) fileReader.close();
            } catch (IOException e) {
                System.out.println("Não conseguiu fechar o arquivo.");
            }
        }
    }
}
```

## 3. A Revolução do `Try-with-Resources` (Java 7+)

O código acima é considerado muito feio e burocrático (o fato de precisar de um try/catch *dentro* do finally para fechar o arquivo é terrível).
A partir do Java 7, introduziram o **Try-with-Resources**.

Se você declarar seus leitores de arquivo **dentro dos parênteses do `try()`**, o Java fecha eles magicamente e automaticamente no final, e você **não precisa mais do bloco `finally`**.

### Novo estilo limpo (Try-with-Resources):
```java
import java.io.*;

public class App {
    public static void main(String[] args) {
        File file = new File("meuarquivo.txt");

        // Declarando as variáveis DENTRO dos parênteses do try
        try (
            FileReader fileReader = new FileReader(file);
            BufferedReader bufferedReader = new BufferedReader(fileReader);
        ) {
            String line = bufferedReader.readLine();
            while (line != null) {
                System.out.println(line);
                line = bufferedReader.readLine();
            }
            // Não precisa de finally! O Java dá .close() automaticamente aqui.
            
        } catch (FileNotFoundException e) {
            System.out.println("Arquivo não encontrado.");
        } catch (IOException e) {
            System.out.println("Problema na leitura.");
        }
    }
}
```

### O Segredo Mágico: Interface `AutoCloseable`
Por que o Try-with-Resources funciona? Porque as classes `FileReader` e `BufferedReader` implementam a interface `AutoCloseable`.
Se você criar a sua própria classe, implementar `AutoCloseable` e programar o método `close()`, a sua classe também será fechada automaticamente se usada dentro de um Try-with-Resources.

## 4. Lançando as Suas Próprias Exceções (`throws`)

Você pode alertar outros programadores que o seu método pode falhar. Se você cria um método que pode dar problema, coloque `throws NomeDaExceção` na assinatura dele.

```java
// O 'throws Exception' alerta que esse método é perigoso
public int subtrair10(int num) throws Exception {
    if (num < 10) {
        // Criando e atirando o erro ativamente
        throw new Exception("O número passado foi menor que 10!");
    }
    return num - 10;
}
```
*   **Dica Profissional:** Não capture `NullPointerException` com try/catch. Esse é um erro de lógica de programação. É papel do desenvolvedor verificar se algo é `null` (usando `if(var == null)`) antes de tentar usar a variável.
