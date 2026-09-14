# Capítulo: Test Driven Development (TDD) em Java

Neste capítulo da transcrição, o instrutor foca na aplicação prática do **Desenvolvimento Orientado a Testes (TDD)**, mostrando como criar uma funcionalidade (um serviço de usuários) guiando o design do código inteiramente através dos testes.

## 1. O Ciclo do TDD
O TDD é uma metodologia de engenharia de software baseada em um ciclo curto de repetições, conhecido como **Red, Green, Refactor, Repeat**:

1.  **Red (Vermelho - Falha):** Você escreve o teste unitário **antes** do código de produção existir. Obviamente, o código nem vai compilar ou o teste vai falhar.
2.  **Green (Verde - Passou):** Você vai no código de produção e escreve a **quantidade mínima** de código necessária apenas para fazer o código compilar e aquele teste específico ficar verde.
3.  **Refactor (Refatorar):** Com a rede de segurança verde garantida, você revisa o código do teste e o código de produção, melhora nomes de variáveis, remove duplicações e aplica padrões de projeto (Design Patterns). O teste deve continuar verde no final.
4.  **Repeat (Repetir):** Repita o processo para a próxima regra de negócio da funcionalidade.

## 2. A Prática: O Teste como o Primeiro "Cliente" da sua API
A regra principal ao codificar em TDD é parar de escrever o teste no momento exato em que ele apontar um erro.
*   **Ação:** O teste tenta instanciar `new UserServiceImpl()`. 
*   **Erro:** A classe não existe. 
*   **Correção:** Pare o teste, vá na pasta main, crie a interface `UserService` e a implementação `UserServiceImpl` (*Programar para Interfaces*).
*   **Ação:** O teste tenta chamar `.createUser(...)`. 
*   **Erro:** O método não existe. 
*   **Correção:** Vá na interface e defina a assinatura do método.
*   **Ação:** O teste tenta validar `user.getFirstName()`. 
*   **Erro:** O método retorna `null`. 
*   **Correção:** Vá na classe e crie a lógica para preencher o atributo.

**Benefício:** Esse processo garante que o seu código de produção só conterá os métodos e lógicas estritamente necessários para satisfazer o sistema. Evita-se a criação de códigos "inúteis" ou métodos que ninguém nunca vai chamar.

## 3. Refatorando o Código de Teste
A etapa de refatoração se aplica muito aos testes. Conforme você escreve novos testes, perceberá que a fase de **Arrange (Preparação)** se repetirá muito.
*   Quando a preparação de um teste (ex: instanciar a classe e criar as Strings padrão) for idêntica à de outro teste, **remova a duplicação** extraindo a lógica comum para um método anotado com **`@BeforeEach`**.
*   Se dois testes validam o exato mesmo caminho de execução e apenas checam variáveis diferentes (um checa se salvou o nome e o outro se salvou o email), é uma boa prática **mesclá-los em um único teste** com múltiplos `assertEquals` para manter a suíte de testes rápida.

## 4. TDD com Cenários Negativos (Exceções)
No TDD, após cobrir o cenário de sucesso (Happy Path), devemos escrever testes que forcem os erros de validação:
1.  **Red:** Escreva um teste passando um nome em branco (`""`) para o método `createUser` e envolva a chamada num `assertThrows(IllegalArgumentException.class, ...)`. O teste falhará, pois o método de produção aceitará o nome em branco sem reclamar.
2.  **Green:** Adicione um bloco `if (firstName.isEmpty()) { throw new IllegalArgumentException("User first name is empty"); }` no código de produção. O teste passará.
3.  **Refactor:** Aproveite que o teste garantiu o comportamento, extraia mensagens de erro para constantes e melhore a validação.

> [!TIP]
> **Conclusão:** O TDD te dá o dobro de confiança. Se você alterar regras complexas no futuro, a sua extensa suíte de testes validará quase instantaneamente se alguma quebra (*Regressão*) ocorreu.
