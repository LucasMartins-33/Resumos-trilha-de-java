# Questões Práticas - Capítulo 08 (Mockito - Testando Código Java em Isolamento)

---

### 🟢 Nível 1: Habilitando o Mockito com JUnit 5
**Cenário:** Você precisa preparar uma classe de teste unitário para utilizar o ecossistema do Mockito no JUnit 5.
**Sua Tarefa:**
* Crie a classe de teste `UsuarioServiceTest`.
* Adicione a anotação `@ExtendWith(MockitoExtension.class)` acima da declaração da classe.
* Importe a extensão de `org.mockito.junit.jupiter.MockitoExtension`.
* Crie um método `@Test` vazio e execute-o para confirmar que a extensão inicializa sem erros.

---

### 🟡 Nível 2: Injetando Dublês com `@Mock` e `@InjectMocks`
**Cenário:** A classe `UsuarioService` depende de `UsuarioRepository` e `EmailService` em seu construtor. Você precisa testar `UsuarioService` de forma isolada.
**Sua Tarefa:**
* Na classe de teste, declare:
  * `@Mock private UsuarioRepository usuarioRepository;`
  * `@Mock private EmailService emailService;`
  * `@InjectMocks private UsuarioService usuarioService;`
* Crie um teste e verifique com `assertNotNull(usuarioService)` se o Mockito instanciou a classe real e injetou os dois dublês automaticamente.

---

### 🟠 Nível 3: Stubbing Básico com `when().thenReturn()`
**Cenário:** O método `usuarioService.buscarPorId(1L)` chama internamente `usuarioRepository.findById(1L)`. Como o repositório é um mock, ele retornará `null` se não for instruído.
**Sua Tarefa:**
* Na fase Arrange do teste, ensine o mock utilizando:
  ```java
  Usuario usuarioSimulado = new Usuario(1L, "Lucas", "lucas@email.com");
  when(usuarioRepository.findById(1L)).thenReturn(Optional.of(usuarioSimulado));
  ```
* Na fase Act, execute `Usuario resultado = usuarioService.buscarPorId(1L);`.
* Na fase Assert, confirme que `resultado.getNome()` é `"Lucas"`.

---

### 🔴 Nível 4: Flexibilizando Entradas com Argument Matchers (`any`)
**Cenário:** O método de salvar usuário recebe instâncias dinâmicas cujas referências de memória variam, impossibilitando igualdade estrita de ponteiros.
**Sua Tarefa:**
* Importe os matchers estáticos de `org.mockito.ArgumentMatchers.*`.
* Configure o mock usando `any(Usuario.class)`:
  ```java
  when(usuarioRepository.save(any(Usuario.class))).thenReturn(usuarioSimulado);
  ```
* Execute o método de negócio e verifique se o retorno é recebido independentemente da instância específica de usuário passada.

---

### 🟣 Nível 5: Verificando Interações com `verify()` e Contagem de Vezes
**Cenário:** Além de checar o retorno, você precisa garantir que o método `save()` do repositório foi de fato chamado exatamente 1 vez durante o fluxo.
**Sua Tarefa:**
* Importe `verify` e `times` de `org.mockito.Mockito.*`.
* Após a fase Act, adicione a verificação:
  ```java
  verify(usuarioRepository, times(1)).save(any(Usuario.class));
  ```
* Tente alterar o valor para `times(2)` e observe o erro detalhado do Mockito reportando *Wanted 2 times but was 1 time*.

---

### 🟤 Nível 6: Assegurando que Métodos NUNCA Foram Invocados (`never()`)
**Cenário:** Se a tentativa de cadastrar um usuário falhar por validação de senha fraca, o serviço JAMAIS deve chamar o repositório para persistir nem enviar e-mail.
**Sua Tarefa:**
* No teste com senha inválida, invoque o método que rejeita o cadastro.
* Utilize o matcher `never()` para comprovar a segurança do sistema:
  ```java
  verify(usuarioRepository, never()).save(any());
  verify(emailService, never()).enviarEmailBoasVindas(any());
  ```

---

### 🔵 Nível 7: Simulando Falhas e Exceções com `thenThrow()`
**Cenário:** Você precisa testar se sua classe de serviço captura uma queda de banco de dados (`DataAccessException`) e a converte em uma exceção de negócio amigável (`ServiceException`).
**Sua Tarefa:**
* Ensine o mock a lançar a exceção:
  ```java
  when(usuarioRepository.save(any())).thenThrow(new RuntimeException("Conexão recusada pelo banco"));
  ```
* Invoque o serviço dentro de um `assertThrows(ServiceException.class, () -> usuarioService.cadastrar(dto));`.
* Valide se a mensagem de erro traduzida atende ao requisito.

---

### 🟢 Nível 8: Tratando Métodos Void com `doThrow()`
**Cenário:** O método `emailService.enviarEmail(...)` tem retorno `void`. A sintaxe `when(emailService.enviarEmail(...))` não compila em Java.
**Sua Tarefa:**
* Utilize a sintaxe invertida do Mockito para métodos `void`:
  ```java
  doThrow(new EmailException("Falha no servidor SMTP"))
      .when(emailService).enviarEmail(any(), any());
  ```
* Execute o fluxo de disparo de e-mail e valide com `assertThrows` se a falha é propagada ou tratada conforme esperado.

---

### 🟡 Nível 9: Anulando Comportamentos com `doNothing()`
**Cenário:** Você configurou uma regra global no `@BeforeEach` para lançar exceção em chamadas de e-mail, mas em um teste específico precisa que o método void execute sem fazer nada.
**Sua Tarefa:**
* No teste específico, sobrescreva a instrução utilizando `doNothing()`:
  ```java
  doNothing().when(emailService).enviarEmail(any(), any());
  ```
* Execute o método de negócio e confirme que ele conclui com sucesso sem disparar exceções.

---

### 🟠 Nível 10: Integração Final (Fluxo Completo de Cadastro e Notificação)
**Cenário:** Você precisa criar uma suíte unitária completa e impecável para o método `UsuarioResponseDTO registrarNovoUsuario(NovoUsuarioRequestDTO dto)` da classe `UsuarioService`.
**Sua Tarefa:**
* Implemente dois testes unitários exaustivos:
  1. **Cenário de Sucesso:**
     * Repositório verifica se o e-mail já existe (retorna falso).
     * Repositório salva a entidade (retorna o usuário com ID gerado).
     * Serviço envia e-mail de confirmação (método void executando normalmente).
     * Valide os dados retornados no DTO com `assertEquals`.
     * Valide com `verify` se o repositório foi chamado 1 vez e o e-mail foi disparado 1 vez.
  2. **Cenário de E-mail Duplicado:**
     * Repositório informa que o e-mail já está cadastrado.
     * Valide que o serviço dispara `UsuarioJaExisteException`.
     * Valide com `verify(..., never())` que o método `save()` e o envio de e-mail JAMAIS foram chamados.
