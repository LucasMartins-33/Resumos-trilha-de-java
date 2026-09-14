# Anotações de Estudo: Java Core - Capítulo 7 (OOP Interview Prep)

> [!NOTE]
> Este é o seu guia definitivo para responder a perguntas técnicas de Orientação a Objetos em entrevistas. As perguntas e respostas foram destiladas e formatadas para garantir que você tenha a resposta ideal na ponta da língua.

---

## 1. Fundamentos da POO

### O que é POO e o que é um Objeto?
* **POO (Programação Orientada a Objetos)**: É um paradigma de programação baseado na ideia de transformar tudo em objetos. Um objeto estrutura um **estado** (dados/atributos) e um **comportamento** (métodos).
* **Objeto**: É um modelo real de uma entidade que ocupa um espaço físico na memória (`Heap`). É uma instância concreta gerada a partir do molde de uma `Class`.

### Quais são os pilares da POO?
1. Abstração
2. Encapsulamento
3. Herança
4. Polimorfismo

### Quais são os Prós e Contras da POO?
* **Prós (Vantagens)**: 
  * *Modularidade*: Se o pneu do carro quebrar, você debuga a classe `Tire`, sem ler o código do sistema inteiro.
  * *Reuso*: Através da herança e composição.
  * *Flexibilidade*: Através do polimorfismo.
  * *Resolução Eficaz*: Divide um problema titânico em "chunks" (classes) menores e solucionáveis.
* **Contras (Desvantagens)**: Curva de aprendizado inicial maior, código-fonte tende a ser mais longo (maior volume de classes), execução ligeiramente mais lenta (muitos ponteiros de memória e chamadas dinâmicas) e exige um planejamento prévio da arquitetura muito mais refinado.

---

## 2. Herança vs Composição e Relações

### "IS-A" (É um) vs "HAS-A" (Tem um)
Em entrevistas, essas duas expressões caem com muita frequência:
* **"IS-A" (É-um)**: Indica **Herança** (`extends`). Ex: Um Cachorro "é um" Animal.
* **"HAS-A" (Tem-um)**: Indica **Composição**. Ex: Uma casa "tem um" banheiro. Um banheiro não é uma casa. Se você só quer reaproveitar código, mas as classes não compartilham um mesmo DNA de linhagem, fuja da herança e use Composição (injete uma classe dentro da outra).

### Qual a diferença entre Associação, Agregação e Composição?
* **Associação**: A forma mais genérica de dizer que objetos interagem entre si.
* **Agregação (Relacionamento Fraco)**: Uma relação de "parte do todo", mas as partes sobrevivem sem o todo. Ex: Pessoa e Grupo. Se o grupo acaba, a pessoa continua existindo na memória.
* **Composição (Relacionamento Forte)**: Uma relação rigorosa onde a "parte" só existe enquanto o "todo" existir. Ex: Carro e Roda/Motor. No código, o objeto "Roda" é instanciado e morre no mesmo ciclo de vida do "Carro". 

---

## 3. Interfaces vs Classes Abstratas

Essa é provavelmente a pergunta mais frequente. O que responder?
* **Classe Abstrata**: Permite compartilhar lógicas concretas (métodos com corpo) e definir variáveis de estado (`protected`, `private`) que os filhos herdarão. Mas você só pode dar `extends` em **uma** classe. É ideal quando você tem uma hierarquia forte onde filhos dependem de um comportamento padrão do pai.
* **Interface**: Define estritamente um contrato de ações (O QUE deve ser feito). Não mantém variáveis de estado (só constantes). Uma classe pode implementar **múltiplas** interfaces. Excelente para criar habilidades genéricas (`Flyable`, `Drivable`).
* **Qual usar?**: Na dúvida, crie a *Interface*. Você pode até mesclar as duas abordagens, criando a Interface, criando a Classe Abstrata para implementar um padrão e fazendo as classes concretas herdarem a classe abstrata.

---

## 4. Cast de Tipos e ClassCastException

> [!WARNING]
> No material enviado ocorreu um corte, mas a explicação técnica vital que completa a seção de *casting* é a seguinte:

* **Automatic/Upcasting (Ampliação)**: Funciona naturalmente. Passar um `byte` para um `int` ou salvar um `Circle` numa variável `Shape`. Não requer conversão explícita.
* **Explicit/Downcasting (Estreitamento)**: É ir do genérico para o específico. Exige colocar os parênteses `(Circle) meuShape`.
* **Quando ocorre o `ClassCastException`?**: Ocorre em *tempo de execução* se você tentar fazer um Downcasting mentiroso. Por exemplo: A variável `Shape` abriga um `Triangle` na memória. Se você tentar forçar `(Circle) meuShape`, o Java perceberá o erro na hora da execução e lançará a exceção. 
  * *Dica de ouro*: Sempre faça a verificação `if (meuShape instanceof Circle)` antes de fazer o downcast!

---

## 5. Questões Rápidas e Dinâmicas

**1. O que é Dynamic Binding?**
R: É a forma como o Polimorfismo funciona. Como o compilador não sabe qual implementação de uma interface foi instanciada até que o programa rode, a decisão de qual método específico invocar (ex: o método `draw()` do triângulo ou do círculo) é amarrada, em tempo de execução, ao objeto real escondido pela variável.

**2. Posso sobrecarregar (overload) um método estático?**
R: Sim, sem problemas. Você pode criar vários métodos estáticos com o mesmo nome e parâmetros diferentes. O que você **não pode** fazer é sobrescrever (override) um método estático via polimorfismo de herança.

**3. O que acontece com a inicialização se eu esquecer o Default Constructor?**
R: O compilador Java cria um construtor vazio "invisível" para você. Vale lembrar: na herança, se a classe filha não usar a palavra `super()` na primeira linha para chamar o pai, o Java tentará fazer a chamada `super()` implicitamente para o construtor vazio do pai. Se o pai não tiver um construtor vazio, você terá um erro de compilação na hora.

**4. Um objeto pode acessar uma variável privada da sua classe?**
R: Sim. Objetos da mesma classe conseguem acessar e enxergar as variáveis privadas de outras instâncias da mesma classe se eles estiverem operando dentro dela, embora no uso cotidiano as manipulemos indiretamente usando Reflection API ou expondo *Getters*.

**5. Quais os tipos de classes internas (Inner Classes) e para que servem?**
R: Elas servem para encapsular lógica auxiliar e permitem contornar a falta de herança múltipla no Java, pois cada classe interna pode herdar implementações diferentes, independente do que a classe pai herda. Existem 4 tipos:
1. *Static nested classes*: Aninhadas estáticas. Não precisam da instância da classe externa para existir. Só acessam métodos estáticos da classe pai diretamente.
2. *Non-static inner classes*: Classes internas comuns. Precisam da instância da classe externa e têm acesso aos atributos não-estáticos dela.
3. *Local classes*: Definidas dentro de um bloco específico (geralmente dentro do corpo de um método).
4. *Anonymous classes*: Classes sem nome definidas e instanciadas no mesmo bloco.

**6. O que é permitido mudar ao sobrescrever (Override) um método?**
R: Em entrevistas, eles sempre tentam te confundir com isso:
* **Modificador de Acesso**: SIM, desde que seja para AMPLIAR a visibilidade (ex: de `protected` para `public`). Nunca reduzir.
* **Tipo de Retorno**: SIM, desde que seja uma conversão de hierarquia para baixo (Downcasting / Tipos Covariantes). Ex: Pai retorna `Number`, o Filho pode retornar `Integer`.
* **Tipo ou Quantidade de Argumentos**: NÃO! Se alterar, deixa de ser Override e vira Overload (sobrecarga).
* **Nome do Argumento**: SIM, nomes de parâmetros não importam para a assinatura.
* **Cláusula Throws**: SIM, você pode remover o `throws` completamente, alterar a ordem ou adicionar novas exceções que sejam filhas das originais (ou de runtime).
* *Pegadinha*: Métodos `private` **nunca** podem ser sobrescritos!
