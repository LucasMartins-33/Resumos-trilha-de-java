# Capítulo 08: Mockito - Testando Código Java em Isolamento

Este capítulo introduz o **Mockito**, um dos frameworks de testes mais populares no mundo Java. Ele permite criarmos objetos "falsos" (Mocks ou Test Doubles) para testarmos nossa lógica de negócio em isolamento, sem depender de banco de dados reais, APIs externas ou componentes de terceiros.

## 1. O que são Mocks e por que usá-los?
Imagine que você está testando um método `createUser()` que, internamente, chama `UserRepository.save()` para gravar no MySQL.
**Em testes unitários puros, você NÃO deve conectar ao banco de dados.** Caso o banco esteja fora do ar, seu teste vai falhar por um erro externo de rede, e não porque a sua lógica está quebrada.

A solução? Substituir o repositório real por um **Mock**. Um Mock permite que você dite as regras: *"Sempre que alguém pedir para salvar, apenas minta dizendo que deu certo e retorne verdadeiro, sem gravar nada no HD"*. Isso nos isola e foca apenas na regra do nosso método `createUser()`.

## 2. Configurando o Mockito
Para habilitar o framework em um projeto, adicione a dependência `mockito-junit-jupiter` no Maven (`pom.xml`) ou Gradle. Depois, anote a sua classe de testes para liberar o suporte:
```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest { ... }
```

## 3. Criando e Injetando Mocks
O Mockito usa anotações muito elegantes para preparar o cenário automaticamente:
*   **`@Mock`**: Cria um objeto "fantasma" / dublê de uma dependência.
*   **`@InjectMocks`**: Instancia a nossa classe real sob teste, rastreando automaticamente todos os `@Mock`s criados e os injetando em seu construtor.

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {

    @Mock
    UserRepository userRepository; // Dependência falsa

    @Mock
    EmailService emailService; // Dependência falsa

    @InjectMocks
    UserServiceImpl userService; // Objeto REAL. O Mockito vai passar os dois mocks ali de cima no construtor dele.
}
```

## 4. Stubbing: Condicionando Comportamentos (`when()`)
Temos que ensinar ao Mockito como reagir quando alguém chamar os métodos falsos. Usamos o padrão estático `when(chamada).thenReturn(resposta)`.

```java
import static org.mockito.Mockito.*;
import static org.mockito.ArgumentMatchers.any;

@Test
void testaCriacao() {
    // Arrange (Ensinando o mock)
    // O any(User.class) avisa que não nos importamos com qual usuário foi passado. Qualquer um serve.
    when(userRepository.save(any(User.class))).thenReturn(true);
    
    // Act
    userService.createUser("Lucas", "Senha123");
}
```

## 5. Verificando Chamadas (`verify()`)
Uma das validações mais poderosas do Mockito. Em vez de verificar apenas o retorno de um método, podemos confirmar se a nossa classe principal pelo menos **tentou** chamar o banco de dados.
```java
// Verifica se o método save() foi chamado EXATAMENTE 1 VEZ durante todo o teste
verify(userRepository, times(1)).save(any(User.class));

// O Mockito disponibiliza várias validações:
verify(userRepository, never()).save(any()); // Certifica de que NUNCA tentaram salvar
verify(userRepository, atLeast(2)).save(any()); // Chamado no mínimo 2 vezes
verify(userRepository, atMostOnce()).save(any()); // Chamado 0 ou 1 vez no máximo
```

## 6. Simulando Exceções
Uma das grandes vantagens dos Mocks é a capacidade de forçar cenários caóticos difíceis de reproduzir no mundo real (como o banco de dados cair ou a API de email devolver timeout).
Usamos o `.thenThrow()`:
```java
when(userRepository.save(any(User.class))).thenThrow(new RuntimeException("Banco Caiu"));

// Nossa classe (userService) vai lidar com esse RuntimeException jogando uma ServiceException? Vamos testar:
assertThrows(UserServiceException.class, () -> userService.createUser(...));
```

## 7. Mockito em Métodos Void (Sem Retorno)
O padrão clássico `when(mock.metodo()).thenReturn(...)` **NÃO COMPILA para métodos `void`** (ex: o `emailService.sendEmail()`). 
Quando o método for `void`, a sintaxe do Mockito se inverte para `doAlgumaCoisa().when(mock).metodoVoid()`:

*   **Forçando Exceção:** `doThrow(new Exception()).when(emailService).sendEmail(any());`
*   **Não faça nada (Anulação):** `doNothing().when(emailService).sendEmail(any());` 
    *(Ideal se no seu `@BeforeEach` você configurou um `doThrow` global, mas num teste específico você quer resetar e permitir que a rotina passe livre).*
*   **Execute a rotina real:** `doCallRealMethod().when(emailService).sendEmail(any());` 
    *(Pula a barreira de proteção do dublê e executa de verdade o que tem dentro do método. É raramente usado em projetos novos, focado mais em código legado com alto acoplamento).*
