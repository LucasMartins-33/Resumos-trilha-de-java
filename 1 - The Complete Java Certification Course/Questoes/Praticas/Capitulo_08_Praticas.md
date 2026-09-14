# Questões Práticas - Capítulo 08 (Object Orientation)

🟢 Nível 1: O Molde (A Classe Base)
Cenário: Você precisa modelar um sistema de zoológico.
Sua Tarefa: 
* Crie uma classe chamada `Animal` contendo 3 variáveis de instância (atributos): `idade` (int), `genero` (String) e `pesoLbs` (int).
* Deixe a classe sem nenhum método por enquanto (apenas os atributos).

🟡 Nível 2: O Construtor
Cenário: A equipe percebeu que, do jeito que está, criar animais exige preencher variável por variável manualmente no main. Queremos criar o animal já inicializado num único comando.
Sua Tarefa: 
* Na classe `Animal` do nível anterior, crie o Construtor público recebendo `idade`, `genero` e `pesoLbs` como parâmetros.
* Use a palavra-chave `this` para atribuir os parâmetros às variáveis da classe.
* Crie um método `public void comer()` que apenas imprime "Comendo...".

🟠 Nível 3: Herança (`extends`)
Cenário: O zoológico tem uma seção focada em pássaros. Os pássaros são animais, mas eles têm lógicas próprias.
Código de Partida:
```java
public class Passaro extends Animal {
    
    // A IDE aponta um erro vermelho nesta classe. Resolva!
    
}
```
Sua Tarefa:
* O erro do compilador ocorre pois `Animal` tem um construtor exigente, e a classe `Passaro` precisa chamá-lo.
* Crie o construtor do `Passaro` com os mesmos 3 parâmetros e utilize o `super()` na primeira linha repassando eles.

🔴 Nível 4: Sobrescrita de Métodos e Encapsulamento
Cenário: Você percebeu que um pássaro tem um estilo de vida mais ativo.
Sua Tarefa:
* Na classe `Passaro`, crie a sobrescrita do método `comer()` (use a anotação `@Override`).
* Dentro dele, imprima "Pássaro bicando sementes...".
* No seu método `main`, instancie o Passaro (com o `new`) e chame `.comer()`. Certifique-se de que ele não usou o "Comendo..." genérico da classe pai.

🟣 Nível 5: O Contrato (Interfaces)
Cenário: Nem todo animal sabe voar (um Pinguim não voa), e criar um método `voar()` no `Animal` poluiria filhos que não voam. Precisamos de um "contrato de habilidade" avulso.
Sua Tarefa:
* Crie uma `public interface Voavel`.
* Defina dentro dela o método abstrato `void voar();` (sem chaves).
* Faça a classe `Passaro` herdar isso assinando o contrato (use `implements Voavel`).
* O Eclipse pedirá que você implemente o método. Crie o corpo do método `voar()` dentro de `Passaro` imprimindo "Pássaro batendo asas no céu!".

🟤 Nível 6: Classes Abstratas
Cenário: O sistema foi revisado, e a diretoria do zoológico decretou: "Ninguém nunca deveria poder instanciar apenas um `new Animal()`. Um animal puramente abstrato não existe de verdade, ele precisa ser uma sub-espécie concreta!".
Sua Tarefa:
* Modifique a assinatura da classe `Animal` adicionando a palavra `abstract`.
* Vá no seu método `main` e tente fazer `Animal a = new Animal(2, "M", 20);`. Observe o erro que o Eclipse/IDE vai dar (Não compilará). Apague ou comente a linha para seguir em frente.

🔵 Nível 7: Métodos Abstratos Forçados
Cenário: A classe abstrata `Animal` permite que criemos métodos sem corpo que devem ser construídos pelos filhos.
Sua Tarefa:
* Dentro de `Animal`, crie a assinatura `public abstract void seMover();`.
* Vá na classe `Passaro`. A IDE dará erro informando que você precisa implementar esse método herdado não resolvido.
* Implemente o método `seMover` na classe `Passaro`, imprimindo "Pássaro pulando galhos".

🟢 Nível 8: Polimorfismo Prático (Variáveis de Referência Genéricas)
Cenário: Vamos usar a mágica do Polimorfismo. Queremos criar pássaros e peixes, mas agrupá-los todos apenas como "Animais" para os tratadores cuidarem sem se importar com a espécie específica.
Código de Partida:
```java
public static void main(String[] args) {
    // 1. Crie uma variável do tipo ANIMAL genérico, mas que guarde um objeto PASSARO.
    Animal pet = new Passaro(3, "Fêmea", 5);
    
    // 2. Chame pet.seMover(); e rode o código.
    // 3. Tente chamar pet.voar(); e explique no comentário por que o compilador não deixou!
}
```
Sua Tarefa: 
* Reproduza o cenário acima. Note que ao instanciar como a variável abstrata pai, perdemos acesso aos métodos que são exclusivos do filho (`voar()`).

🟡 Nível 9: Polimorfismo através de Interfaces
Cenário: Vamos ignorar a relação de herança genética (Animais) e pensar em aviões. Aviões e Pássaros não são animais em comum, mas AMBOS sabem voar.
Sua Tarefa:
* Crie uma classe `Aviao implements Voavel`. Defina o método `voar()` dela ("Motores a jato acionados!").
* No método `main`, crie uma variável cujo tipo seja a interface: `Voavel v = new Aviao();`. Chame `v.voar();`. 
* Mude o objeto para `Voavel v = new Passaro(...)` e chame `v.voar()`. A magia da abstração permite a mesma variável comandar sistemas biológicos e mecânicos pela mesma interface.

🟠 Nível 10: O Injetor de Ações (Polimorfismo por Parâmetro)
Cenário: Você está programando o simulador central da Natureza. E quer mandar entidades voarem.
Sua Tarefa:
* Crie um método utilitário ESTÁTICO (na classe Main) chamado `public static void decolar(Voavel entidade)`.
* Dentro dele, tudo o que o código faz é chamar `entidade.voar()`.
* No `main`, passe tanto a sua instância do seu `Passaro` quanto a instância do seu `Aviao` como argumento para esse mesmo método `decolar()`. 
* Rode o código. O Java irá decidir em tempo de execução qual classe chamar, mesmo as duas rodando no mesmo método central!
