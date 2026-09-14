📘 Capítulo 4: Princípios SOLID

**O Cenário:**
A classe abaixo viola terrivelmente os princípios do SOLID. Ela foi feita às pressas.

```java
public class FuncionarioService {
    public void calcularSalario(String tipo, double horas) {
        if(tipo.equals("CLT")) {
            System.out.println(horas * 50);
        } else if(tipo.equals("PJ")) {
            System.out.println(horas * 80);
        }
    }
    
    public void salvarNoBanco(String nome) {
        System.out.println("Salvando " + nome + " no SQL Server...");
    }
}
```

Sua missão é refatorar o código resolvendo as atividades abaixo:

🟢 Atividade 4.1: Single Responsibility Principle (SRP) - Parte 1
1. Identifique as duas responsabilidades distintas dentro de `FuncionarioService`.
2. Crie uma nova classe `FuncionarioRepository`.
3. Mova o método `salvarNoBanco` para essa nova classe.

🟢 Atividade 4.2: Single Responsibility Principle (SRP) - Parte 2
1. Remova o método de banco de dados da `FuncionarioService`.
2. Agora a classe tem apenas a responsabilidade de regras de negócio de salário.

🟡 Atividade 4.3: Open/Closed Principle (OCP) - O Problema
1. Perceba que se a empresa contratar estagiários ("ESTAGIO"), você terá que alterar a classe `FuncionarioService`, quebrando o OCP.
2. Crie uma interface `CalculadoraSalario` com o método `double calcular(double horas);`.

🟡 Atividade 4.4: Open/Closed Principle (OCP) - A Solução
1. Crie as classes `CalculadoraCLT` e `CalculadoraPJ` implementando a interface.
2. Refatore o método `calcularSalario` para receber a interface ao invés da `String tipo`, removendo todos os `if/else`.

🟠 Atividade 4.5: Liskov Substitution Principle (LSP) - Cenário
Imagine uma classe `Passaro` com método `voar()`. A classe `Pinguim` herda de `Passaro` e lança uma exceção no método `voar()`.
1. Crie essas classes e escreva um teste simples.
2. Observe que substituir um `Passaro` por um `Pinguim` quebra a aplicação.

🟠 Atividade 4.6: Liskov Substitution Principle (LSP) - Correção
1. Remova o método `voar()` da classe `Passaro` (deixando apenas coisas genéricas como `comer()`).
2. Crie uma interface separada `AveVoadora` com o método `voar()`.
3. Faça apenas pássaros que voam (ex: `Aguia`) implementarem essa interface.

🔴 Atividade 4.7: Interface Segregation Principle (ISP) - Cenário
1. Crie a interface `Trabalhador` com os métodos `trabalhar()` e `receberBeneficioValeAlimentacao()`.
2. Crie a classe `RoboIndustial` implementando essa interface.
3. Perceba que o robô é forçado a implementar `receberBeneficioValeAlimentacao()`, violando o ISP.

🔴 Atividade 4.8: Interface Segregation Principle (ISP) - Correção
1. Divida a interface `Trabalhador` em duas: `TrabalhadorOperacional` e `TrabalhadorComBeneficios`.
2. Ajuste a classe `RoboIndustial` para implementar apenas a interface correta.

🔴 Atividade 4.9: Dependency Inversion Principle (DIP) - Problema
1. Volte à classe `FuncionarioService` (Atividade 4.2).
2. Se você instanciar diretamente `FuncionarioRepository repo = new FuncionarioRepository();` dentro dela, você está acoplado a uma implementação de baixo nível.

🔴 Atividade 4.10: Dependency Inversion Principle (DIP) - Solução (Injeção)
1. Transforme `FuncionarioRepository` numa interface.
2. Crie `SqlFuncionarioRepository` implementando ela.
3. Injete a interface no construtor de `FuncionarioService`. Agora o serviço alto nível depende de abstração!
