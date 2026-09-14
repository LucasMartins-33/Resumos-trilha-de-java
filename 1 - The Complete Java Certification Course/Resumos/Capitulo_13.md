# Capítulo 13: Advanced Concepts & Deploying Java Programs (JAR Files)

Neste capítulo, expandimos o conhecimento para fora da IDE (Eclipse), entendendo como o código é compilado, empacotado e rodado via linha de comando, além de abordar três conceitos vitais de manipulação de objetos avançada: Clonagem, Comparação e Serialização.

## 1. Clonagem de Objetos (`Cloneable`)

Para criar uma cópia idêntica de um objeto sem usar a palavra `new` (o que consome muito processamento), usamos o método `.clone()`.

**Regras para Clonagem:**
1. A classe deve implementar a interface marcação `java.lang.Cloneable` (se não o fizer, o Java lançará `CloneNotSupportedException`).
2. O método `.clone()` original vem da superclasse `Object` e é `protected`, então você deve sobrescrevê-lo na sua classe para torná-lo `public` e usável.

**Sintaxe de Exemplo:**
```java
class Student implements Cloneable {
    int rollno;
    String name;

    // Construtor normal
    Student(int rollno, String name) {
        this.rollno = rollno;
        this.name = name;
    }

    // Método que permite a clonagem
    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

// Uso:
Student s1 = new Student(101, "Amit");
Student s2 = (Student) s1.clone(); // Cria uma cópia exata do s1
```

## 2. Interface Comparable (`Comparable`)

Se você tem uma lista de `Student` e tenta usar `Collections.sort(lista)`, o Java não saberá se deve ordená-los por nome, por idade, ou por nota. 
A interface `Comparable` ensina o seu objeto a se **auto-ordenar** (Ordem Natural).

**Regras do Comparable:**
1. Implemente `Comparable<SuaClasse>`.
2. Sobrescreva o método `compareTo()`.
3. O método deve retornar: 
   * `0` se forem iguais.
   * `Número positivo (1)` se o objeto atual for maior que o recebido.
   * `Número negativo (-1)` se o objeto atual for menor.

```java
class Student implements Comparable<Student> {
    int age;
    
    @Override
    public int compareTo(Student st) {
        if (this.age == st.age) return 0;
        else if (this.age > st.age) return 1;
        else return -1;
    }
}
```

## 3. Serialização de Objetos (`Serializable`)

Serializar é o processo de pegar um Objeto Vivo na memória RAM e transformá-lo em uma **sequência de bytes**. Esses bytes podem ser salvos em um arquivo no HD (`.ser`), enviados por uma rede ou banco de dados. 

**Regras da Serialização:**
1. A classe deve implementar `java.io.Serializable`.
2. Todos os atributos da classe precisam ser serializáveis.
3. Se você não quiser que uma variável específica seja salva/enviada (por exemplo, uma senha), coloque a palavra `transient` nela.

* **Serializar (Salvar):** Utiliza-se `ObjectOutputStream` e `.writeObject(obj)`.
* **Deserializar (Ler):** Utiliza-se `ObjectInputStream` e `.readObject()`.

## 4. O Coração do Java: Compilando via Linha de Comando (CMD)

O Java não lê seu arquivo `.java`. Ele lê um arquivo `.class` (Bytecode). A IDE faz isso por baixo dos panos, mas na unha funciona assim:

1. **Escrever (Human Readable):** Você escreve o código no arquivo `Program.java`.
2. **Compilar (Javac):** No terminal, você roda:
   `javac Program.java`
   *Isso traduz o código humano e gera um arquivo `Program.class` cheio de caracteres ilegíveis que a máquina virtual (JVM) entende.*
3. **Executar (Java):** No terminal, você roda o arquivo recém-criado (sem a extensão):
   `java Program`
   *Isso dá vida ao seu programa no console.*

## 5. Deployment: Criando e Rodando Arquivos JAR

Um **JAR (Java ARchive)** é literalmente um "zip" do mundo Java. Ele empacota dezenas, centenas de arquivos `.class` dentro de um único arquivo executável (ex: `MeuJogo.jar`). 

Para o sistema saber qual classe ele deve executar primeiro (onde está o método `main`), precisamos de um arquivo chamado `manifest.mf`.

**Como montar um arquivo Manifest (`manifest.mf`):**
```text
Main-Class: ExampleProgram
(Dê um Enter para a linha 2, é obrigatório)
```

**Principais Comandos JAR no Terminal:**
*   **Criar (Create):** `jar -cvfm meu_app.jar manifest.mf *.class`
    *(c=create, v=verbose, f=file, m=manifest. Ele junta o manifest com todos os arquivos class no app.jar).*
*   **Visualizar Conteúdo (Table of Contents):** `jar -tvf meu_app.jar`
*   **Extrair (Extract):** `jar -xvf meu_app.jar`
*   **Rodar o Programa Final:** `java -jar meu_app.jar`
