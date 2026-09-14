# Questões Práticas - Capítulo: Test Driven Development (TDD) em Java

---

### 🟢 Nível 1: A Fase Vermelha (Red) - O Teste Antes do Código Existir
**Cenário:** Você precisa implementar um serviço que calcula o desconto previdenciário de funcionários, mas a classe ainda não existe no projeto.
**Sua Tarefa:**
* Crie a classe de teste `CalculadoraPrevidenciariaTest` em `src/test/java`.
* Escreva um método de teste que tenta instanciar `new CalculadoraPrevidenciaria()` e chamar `.calcularDesconto(3000.0)`.
* Observe o compilador do Java acusar erro vermelho: a classe e o método não existem.
* Entenda por que no TDD esse erro de compilação representa o primeiro passo formal do ciclo (**Red**).

---

### 🟡 Nível 2: A Fase Verde (Green) - A Quantidade Mínima de Código
**Cenário:** Com o teste vermelho gerado no Nível 1, sua missão agora é fazê-lo passar o mais rápido possível, sem inventar regras complexas prematuras.
**Sua Tarefa:**
* Vá para `src/main/java` e crie a classe `CalculadoraPrevidenciaria`.
* Crie o método `public double calcularDesconto(double salario)`.
* Faça o método retornar o valor exato fixo exigido pelo teste (por exemplo, `return 330.0;` caso o teste espere 330.0).
* Execute o teste e observe a transição imediata para a **Barra Verde (Green)**.

---

### 🟠 Nível 3: A Fase de Refatoração (Refactor)
**Cenário:** Seu teste está verde, mas o código de produção possui um valor fixo engessado (*hardcoded*). Agora é hora de introduzir a lógica real com segurança.
**Sua Tarefa:**
* Substitua o retorno fixo pelo cálculo real: `return salario * 0.11;`.
* Reexecute o teste e certifique-se de que a barra permanece verde sem quebras.
* Refatore nomes de variáveis para deixar o código autoexplicativo, sabendo que o teste protege você contra regressões.

---

### 🔴 Nível 4: O Teste como o Primeiro "Cliente" da sua API
**Cenário:** Você precisa criar uma funcionalidade para registrar novos clientes e quer desenhar a assinatura do método pensando na ergonomia de quem vai utilizá-lo.
**Sua Tarefa:**
* Escreva o teste `testRegistrarCliente()`.
* No teste, defina como você gostaria de interagir com o método: você prefere passar 7 parâmetros soltos `(nome, sobrenome, cpf, rua, numero, cep, telefone)` ou passar um único objeto agrupador `(ClienteRequestDTO dto)`?
* Use a perspectiva do teste para definir uma assinatura limpa e intuitiva antes de escrever qualquer linha da implementação.

---

### 🟣 Nível 5: Programando para Interfaces (Program to an Interface)
**Cenário:** Ao aplicar TDD para construir o serviço de usuários `UserService`, você quer desacoplar o contrato da implementação concreta.
**Sua Tarefa:**
* Na fase Red, faça o teste instanciar a classe concreta atribuindo-a a uma referência de interface:
  `UserService service = new UserServiceImpl(repositoryMock);`.
* Crie a interface `UserService` com a assinatura `UserRest createUser(UserDetailsRequestModel details);`.
* Crie a classe concreta `UserServiceImpl implements UserService` com o código mínimo para satisfazer o teste.

---

### 🟤 Nível 6: Combatendo Código Especulativo (Princípio YAGNI)
**Cenário:** Durante o desenvolvimento de um método de autenticação, você pensa em implementar suporte a login social, autenticação por biometria e múltiplos fatores (MFA).
**Sua Tarefa:**
* Explique a regra fundamental do TDD e do princípio YAGNI (*You Aren't Gonna Need It*): por que você NÃO deve escrever essas funcionalidades adicionais agora se não existe nenhum teste exigindo-as?
* Escreva estritamente o teste para o requisito atual (login básico por e-mail e senha) e mantenha a implementação focada e enxuta.

---

### 🔵 Nível 7: Refatorando o Código de Teste (Eliminando Duplicação na Preparação)
**Cenário:** Você escreveu 4 testes unitários e notou que em todos eles você instancia o mesmo DTO e os mesmos objetos auxiliares com 10 linhas repetidas.
**Sua Tarefa:**
* Identifique a duplicação na etapa de preparação (*Arrange*).
* Crie um método `@BeforeEach void setUp()` na classe de teste.
* Mova as instanciações comuns para esse método de configuração.
* Limpe os 4 métodos de teste, deixando em cada um apenas a ação específica e a asserção.

---

### 🟢 Nível 8: Agrupando Asserções Redundantes
**Cenário:** Você tem dois testes separados: um chamado `testUsuarioCriado_DeveSalvarNome()` e outro chamado `testUsuarioCriado_DeveSalvarEmail()`. Ambos executam exatamente o mesmo fluxo com os mesmos dados.
**Sua Tarefa:**
* Analise por que manter dois testes que percorrem o mesmo caminho apenas para checar atributos diferentes atrasa a suíte desnecessariamente.
* Mescle as verificações em um único teste coeso `testCriarUsuario_ComDadosValidos_DeveRetornarUsuarioCadastrado()` contendo múltiplos `assertEquals` para conferir tanto o nome quanto o e-mail retornado.

---

### 🟡 Nível 9: TDD com Cenários Negativos (Ciclo Red-Green para Exceções)
**Cenário:** A regra de negócio exige que o método `createUser` rejeite nomes de usuário vazios com uma `IllegalArgumentException`.
**Sua Tarefa:**
* **Passo 1 (Red):** Escreva o teste passando um nome vazio `""` e envolva a chamada no `assertThrows(IllegalArgumentException.class, () -> userService.createUser(dto));`. Execute e veja o teste falhar (pois o método ainda aceita nomes vazios).
* **Passo 2 (Green):** No código de produção, adicione a barreira mínima:
  `if (dto.getFirstName().trim().isEmpty()) throw new IllegalArgumentException("O nome não pode ser vazio");`.
  Execute e veja o teste ficar verde.
* **Passo 3 (Refactor):** Extraia a mensagem de erro para uma constante de domínio e refatore o validador.

---

### 🟠 Nível 10: Integração Final (Ciclo Completo de TDD para um Carrinho de Compras)
**Cenário:** Você precisa construir uma classe de domínio `CarrinhoDeCompras` utilizando rigorosamente a metodologia TDD do primeiro ao último passo.
**Sua Tarefa:**
* Aplique o ciclo Red-Green-Refactor-Repeat para desenvolver, passo a passo:
  1. **Ciclo 1:** Carrinho recém-criado deve ter total de itens igual a 0 e valor total igual a 0.0.
  2. **Ciclo 2:** Adicionar um item com preço R$ 50,00 deve atualizar o total de itens para 1 e o valor para 50.0.
  3. **Ciclo 3:** Adicionar um item com valor negativo ou nulo deve lançar `ItemInvalidoException`.
  4. **Ciclo 4:** Se o valor total do carrinho ultrapassar R$ 200,00, aplicar automaticamente 10% de desconto no método `calcularTotalComDesconto()`.
* Ao final de cada ciclo, execute a suíte, garanta a **Barra Verde** e só então avance para o próximo requisito.
