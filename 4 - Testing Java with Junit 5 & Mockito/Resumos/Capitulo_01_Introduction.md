# Capítulo 01: Introduction (Introdução ao JUnit 5 e Testes Unitários)

Este é o resumo do Capítulo 01, elaborado a partir das transcrições das aulas e complementado com informações atualizadas sobre as boas práticas de desenvolvimento em Java.

## 1. O que é um Teste Unitário?
Um teste unitário é um método pequeno e isolado criado para testar uma pequena parte do seu código (geralmente um único método).
Ele foca em testar **apenas uma única funcionalidade de cada vez** e roda de forma extremamente rápida, pois dependências externas são "falsificadas" (mockadas).

Geralmente, os testes unitários seguem a estrutura **AAA (Arrange, Act, Assert)**:
*   **Arrange (Preparar):** Fase onde você configura as dependências, instancia a classe a ser testada e prepara os dados de entrada.
*   **Act (Agir):** A execução do método que está sendo testado. O resultado é armazenado em uma variável.
*   **Assert (Afirmar/Validar):** Verificação de que o resultado gerado corresponde ao esperado.

### 📝 Sintaxe: Exemplo de um Teste Unitário com JUnit 5
Abaixo está a sintaxe moderna (usando JUnit Jupiter) que você usará frequentemente:

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.assertEquals;

class CalculatorTest {

    @Test // Esta anotação informa ao JUnit que este método é um teste
    void testIntegerDivision() {
        // Arrange
        Calculator calc = new Calculator();
        
        // Act
        int result = calc.integerDivision(4, 2);
        
        // Assert
        // Parâmetros do assertEquals: valor esperado, valor atual, mensagem de erro opcional
        assertEquals(2, result, "4 / 2 deveria resultar em 2"); 
    }
}
```

## 2. Por que escrever Testes Unitários?
*   **Prevenção de Regressão:** Garante que alterações ou refatorações futuras não quebrem funcionalidades antigas. (Regressão é quando um bug novo aparece em um código que antes funcionava).
*   **Cobertura de Casos:** É muito mais fácil rodar baterias de testes com dezenas de valores válidos e inválidos em um teste do que testar a aplicação inteira manualmente toda vez.
*   **Documentação Viva:** Um bom teste unitário mostra na prática como outras partes do código devem interagir com a sua classe.

## 3. O Princípio F.I.R.S.T.
Acrônimo que rege as boas práticas de construção de testes de qualidade:
*   **Fast (Rápido):** Devem rodar em poucos milissegundos. Não acessam banco de dados, arquivos ou rede.
*   **Independent (Independente):** Um teste não pode depender do resultado, ordem de execução ou estado gerado por outro teste.
*   **Repeatable (Repetível):** O teste deve gerar o mesmo resultado sempre (verde ou vermelho), em qualquer máquina e ambiente.
*   **Self-validating (Autovalidável):** O teste deve aprovar ou falhar sozinho, indicando o resultado por conta própria, sem intervenção humana.
*   **Thorough / Timely (Abrangente / Oportuno):** Devem cobrir cenários de sucesso (happy path), cenários de falha e condições de contorno (valores mínimos/máximos). Preferencialmente, escritos no momento de criação da funcionalidade (TDD).

## 4. Testando Código em Isolamento e Injeção de Dependência
Para que os testes sejam rápidos e não dependam de partes do sistema fora do escopo, precisamos testar a classe isoladamente.
*   Se a `Classe A` for testada e depender da `Classe B`, instanciar a `Classe B` dentro da `A` gera uma dependência forte.
*   **Injeção de Dependências (Dependency Injection):** Em vez de instanciar a dependência internamente, passamos a dependência para a classe via construtor ou método (frameworks como Spring fazem isso magicamente).
*   **Mocks/Stubs:** Nos testes, substituímos essas injeções de instâncias reais (que iriam até o banco, por exemplo) por objetos de mentira (mocks). Um framework bastante utilizado em conjunto com JUnit 5 para isso é o **Mockito**.

## 5. A Pirâmide de Testes (Testing Pyramid)
Ao planejar a cobertura de testes da aplicação, os tipos de testes são vistos como uma pirâmide:
*   **Base - Testes Unitários (Unit Tests):** Ficam no fundo. São numerosos, altamente focados, rodam super rápido e são baratos.
*   **Meio - Testes de Integração (Integration Tests):** Quantidade intermediária. Testam a comunicação da aplicação com sistemas externos (como a leitura real em um Banco de Dados MySQL/Postgres ou chamadas HTTP para outra API). São mais lentos que os unitários.
*   **Topo - Testes Ponta a Ponta (End-to-End / E2E):** Ficam no topo da pirâmide. Representam fluxos reais de uso, navegando na interface web ou emulando cliques e navegações completas (usando Selenium/Cypress, por exemplo). Em menor quantidade por serem lentos e custarem mais caro de manter.

## 6. O que é o JUnit 5?
O **JUnit 5** foi criado do zero e possui uma arquitetura modularizada em três componentes diferentes, ao contrário do clássico JUnit 4 que era um bloco só:
1.  **JUnit Platform:** É o coração do ecossistema. A base que provê a infraestrutura e a Test Engine para rodar o teste na Máquina Virtual Java (JVM). É o que o IntelliJ, Eclipse, Maven e Gradle chamam por trás dos panos.
2.  **JUnit Jupiter:** É o modelo de programação, ou seja, é a API moderna que usamos para escrever nossos testes e extensões. Tudo que começamos com `import org.junit.jupiter.*` (como o `@Test`) pertence aqui.
3.  **JUnit Vintage:** Uma "engine" de retrocompatibilidade. Garante que projetos legados que ainda possuem código em JUnit 3 ou JUnit 4 possam continuar rodando sem quebrar caso migrem a base para o JUnit 5.

## 7. Build Tools: Maven e Gradle
O JUnit 5 roda nativamente em ferramentas de build como **Maven** (`pom.xml`) e **Gradle** (`build.gradle`). São essas ferramentas as responsáveis por baixar a dependência (os .jar) do JUnit 5 e embutir no seu projeto, simplificando a configuração.
