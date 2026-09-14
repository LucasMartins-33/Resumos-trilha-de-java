# Questões Práticas - Capítulo 19 (Java Generics)

🟢 Nível 1: A Prática Defasada (Raw Types)
Cenário: Você resgatou um código de 2004 para dar manutenção, e ele parou de compilar direito em versões novas.
Código de Partida:
```java
ArrayList caixa = new ArrayList();
caixa.add("DocumentoSecreto");
caixa.add(12345); 

// Tentar recuperar a primeira string:
String resgate = caixa.get(0); // <-- O Java sublinha de vermelho e bloqueia!
```
Sua Tarefa:
* O Java atual barra aquela extração porque, para ele, a Raw Type cospe `Object`s abstratos indecifráveis.
* Faça o "Downcasting" ou *Cast* explícito, colocando `(String)` antes da chamada de `.get(0)` para prometer ao Java que você jura que aquilo é uma String, consertando o erro sem usar Generics (para simular a programação arcaica).

🟡 Nível 2: Modernizando com Type Safety
Cenário: Vamos atualizar o código de 2004! O chefe não quer gambiarras e *casts* explícitos que podem quebrar.
Sua Tarefa:
* Reescreva a primeira linha, instanciando o ArrayList usando Generics focado **exclusivamente** na classe `String`.
* Note que a linha `caixa.add(12345);` passará a explodir instantaneamente na IDE, salvando o desenvolvedor de um bug. Apague ela.
* Remova o *casting* imposto no Nível 1. A tipagem garante a saída fluida!

🟠 Nível 3: O Molde Genérico Caseiro
Cenário: Chega de depender do ArrayList. Queremos criar nosso próprio contêiner de armazenamento dinâmico!
Sua Tarefa:
* Crie a estrutura física de uma `class PortaJoias<J>`.
* Coloque nela uma variável protegida de estado interno do tipo misterioso genérico `J` (ex: `J itemUnico;`).
* Crie o construtor da classe recebendo uma variável desse mesmo tipo `J` e acoplando à variável de estado interno.
* Crie um método `public J retirarItem()` que devolve a joia de volta. 

🔴 Nível 4: A Dança dos Múltiplos Tipos Genéricos
Cenário: Você está montando um framework estilo *Map* do Java para parear Chaves e Valores.
Sua Tarefa:
* Crie uma `class ParFormato<K, V>`.
* Exija que o construtor dessa classe seja alimentado obrigatoriamente por ambos os dois tipos, guardando-os.
* Vá para a classe `Main`. Dê o `new` para instanciar a sua nova classe `ParFormato` e defina (usando as chaves angulares `<>`) que a chave deve ser String e o valor será Double. 

🟣 Nível 5: O Método Mutante (Métodos Genéricos soltos)
Cenário: Nós não temos tempo para construir uma classe inteira. Precisamos apenas de UM método independente para imprimir a matriz, mas queremos que ele imprima matrizes de Int, de String e de Objetos indistintamente.
Sua Tarefa:
* Crie o método estático avulso fora de qualquer classe genérica. 
* A sintaxe deve declarar o parâmetro `T` antes do void: `public static <T> void imprimirArray(T[] array)`.
* Crie um `for-each` usando a variável mística `T` para iterar, varrer e imprimir todos os dados que chegarem por esse portal, sem dar tela azul.

🟤 Nível 6: O Choque de Realidade do Polimorfismo Falso
Cenário: "O Gerente é um Funcionário, mas a Tropa de Gerentes não é Tropa de Funcionários!".
Sua Tarefa: (Mental e Teórica)
* Imagine que você tentou executar: `List<Funcionario> equipe = new ArrayList<Gerente>();`
* O Java te deu um soco. Você quer muito que a variável `equipe` sirva para absorver a criação tanto do Array de Gerentes quanto do Array dos Estagiários dependendo da ocasião e humor do CEO. Como resolvemos esse bloqueio? Passe para o Nível 7 para implementar a cura.

🔵 Nível 7: A Cura Sem Regras (O Curinga `<?>`)
Cenário: Continuando o caos do nível 6, você cansou de ser bloqueado.
Sua Tarefa:
* Crie um método utilitário `public void fazerAuditoria(List<?> equipeMisturada)`.
* O símbolo do curinga vai relaxar as leis duras do Generic. No corpo dele, tente fazer um `for-each` para ler as coisas. Qual será o TIPO da variável do laço que o Java te forçará a usar já que ele não sabe o que é? (Resposta: A superclasse `Object`).

🟢 Nível 8: O Teto Salva Vidas (`? extends X`)
Cenário: O Curinga selvagem `<?>` não ajudou muito, pois perdemos o método de `.receberSalario()` já que tudo lá agora parece um mero `Object` aos olhos do compilador.
Sua Tarefa:
* Altere a declaração de método do nível 7 para fixar o teto superior limitante do curinga.
* Assinatura: `public void pagarTodos(List<? extends Funcionario> listaGeral)`.
* Deste momento em diante, no loop, o Java voltará a permitir e forçar você a usar a variável como o Tipo Teto `Funcionario`. Todos os gerentes (filhos) que passarem e baterem a cabeça nesse teto, cederão e permitirão a execução suave do método `.receberSalario()` de modo polimórfico universal.

🟡 Nível 9: O Chão Sólido (`? super X`)
Cenário: Você precisa de um método para empurrar novos Estagiários para uma caixa. Para proteger a integridade corporativa, essa caixa que virá da Main deve ser focada apenas em níveis baixíssimos, sendo proibida a injeção da lista Master da base de `Funcionarios`.
Sua Tarefa:
* O uso de restrições por piso (Lower Bounds) é raro, mas letal. 
* Defina a assinatura exótica de um método restritivo: `void adicionarNaBase(List<? super Estagiario> caixaDaBase)`.

🟠 Nível 10: Integração Final (O Mestre dos Generics)
Cenário: Uma classe genérica restritiva que funde Classes Customizadas com Curingas! 
Sua Tarefa: 
* Desenhe a classe suprema: `class JaulaDeAnimaisSelvagens<T extends AnimalSelvagem>`.
* Ao exigir o extends logo na declaração primária de classe, o arquiteto inviabilizou para sempre qualquer desenvolvedor inexperiente de instanciar `new JaulaDeAnimaisSelvagens<CachorroDomestico>()`! Essa é a beleza indomável da validação de arquiteturas profundas via Generics. Apenas instancie um caso que passaria no crivo desse portal.
