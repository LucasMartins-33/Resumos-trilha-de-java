# Questões Práticas - Capítulo 16 (The Collections Framework)

🟢 Nível 1: A Lista Dinâmica Polimórfica (ArrayList)
Cenário: Crie o inventário inicial do seu personagem do RPG.
Sua Tarefa: 
* Declare a variável usando a interface genérica `List` (do tipo genérico `String`) e instancie-a como um `ArrayList`.
* Adicione as Strings: "Espada", "Escudo", "Poção".
* Crie um laço `for-each` para imprimir todos os itens da lista.

🟡 Nível 2: Limpando os Duplicados (Set)
Cenário: Nosso sistema de rifas teve um erro no frontend e a pessoa comprou o mesmo bilhete de número `77` três vezes.
Sua Tarefa:
* Declare uma variável `Set<Integer>` instanciada como um `HashSet`.
* Adicione com `.add()` os números 12, 45, 77, 77 e 77.
* Imprima a quantidade de bilhetes ativos usando o `.size()` do Set. (Note que deve dar 3 e não 5).

🟠 Nível 3: O Problema do Equals no Set
Cenário: Mesmo usando Set, clones estão invadindo o banco de dados.
Código de Partida:
```java
class Funcionario {
    String cpf;
    public Funcionario(String cpf) { this.cpf = cpf; }
}

public static void main(String[] args) {
    Set<Funcionario> empresa = new HashSet<>();
    empresa.add(new Funcionario("123.456"));
    empresa.add(new Funcionario("123.456")); // <-- Deveria bloquear, mas o Java deixa passar!
}
```
Sua Tarefa:
* O Java acha que são dois funcionários diferentes pois nasceram de `new` diferentes. Na sua mente (ou num comentário do código), responda quais são os DOIS métodos herdados da classe mãe genérica `Object` que precisamos sobrescrever urgentemente na classe `Funcionario` para forçar o Set a barrar cpfs iguais.

🔴 Nível 4: A Lista Ligada (LinkedList)
Cenário: Um sistema de chat onde mensagens entram no início e no final incessantemente.
Sua Tarefa:
* Instancie uma `LinkedList<String>`.
* Diferente do ArrayList, ela possui métodos exclusivos para manipular as "pontas" dos vagões.
* Adicione um item normal com `.add("Meio")`.
* Utilize o método específico do LinkedList `.addFirst("Início")` e `.addLast("Fim")`.

🟣 Nível 5: Ordem Natural Obrigatória (TreeSet)
Cenário: Você quer guardar números que o usuário digita e quer que o Java SEMPRE os mantenha em ordem crescente automaticamente, sem você precisar chamar métodos de ordenação.
Sua Tarefa:
* Inicialize um `Set<Integer>` com o tipo especial `TreeSet`.
* Adicione números caóticos: 99, 1, 50, 4, 1.
* Faça um print completo da variável Set. Verifique se o resultado saiu como [1, 4, 50, 99].

🟤 Nível 6: Ordenando Objetos Customizados (Comparable)
Cenário: O RH mandou um ArrayList de `Funcionario` (do Nível 3). Mas pediu para você imprimir em ordem alfabética do nome deles.
Sua Tarefa:
* Use a classe utilitária: `Collections.sort(lista)`.
* Para ela parar de ficar vermelha, abra a classe Funcionario e adicione: `implements Comparable<Funcionario>`.
* Sobrescreva o `compareTo`. Dica bônus: como você quer comparar uma String (cpf/nome) com outra, você pode simplesmente usar o `compareTo` já nativo da classe String, assim: `return this.nome.compareTo(outro.nome);`.

🔵 Nível 7: O Cofre de Senhas (HashMap)
Cenário: Um dicionário de perfis.
Sua Tarefa:
* Instancie um `Map<String, String>` (Chave é CPF, Valor é a Senha) usando um `HashMap`.
* Adicione o par: CPF "111", senha "123". Use o método `.put()`.
* Adicione o par: CPF "222", senha "abc".
* Resgate e imprima a senha do CPF "222" usando o método `.get("222")`.

🟢 Nível 8: A Sobrescrita Destrutiva
Cenário: O usuário do CPF "111" redefiniu a senha.
Sua Tarefa:
* Usando o código do Nível 7, dê um novo comando de `.put()` passando a MESMA chave "111", mas enviando a senha nova "999".
* Verifique o tamanho do dicionário usando `.size()`. Ele NÃO deve ter aumentado para 3 pares.
* Dê um `.get("111")` e confirme a alteração.

🟡 Nível 9: Percorrendo o Map (EntrySet)
Cenário: O administrador quer ver toda a tabela de senhas. O Map não suporta um `for` normal.
Sua Tarefa:
* Crie o loop `for-each` avançado do Map.
* A sintaxe é: `for (Map.Entry<String, String> linha : mapa.entrySet()) { ... }`.
* Imprima a chave e a senha na mesma linha usando `.getKey()` e `.getValue()`.

🟠 Nível 10: O Ecossistema Real (Maps Aninhados com Listas)
Cenário: Você está estruturando os dados de um App de Músicas (estilo Spotify).
Sua Tarefa (Avançado - Apenas desenhe a estrutura e instancie):
* Crie um Map onde a **Chave** é o Nome do Artista (String) e o **Valor** é uma LISTA Inteira com o nome das músicas (`List<String>`).
* Instancie: `Map<String, List<String>> acervo = new HashMap<>();`.
* Crie uma Lista para os Beatles, adicione duas músicas e faça um `.put("Beatles", listaBeatles)` no mapa mestre. Crie para outro artista e insira. Isso demonstra o poder sem limites do Collections Framework em hospedar bases de dados inteiras na memória RAM.
