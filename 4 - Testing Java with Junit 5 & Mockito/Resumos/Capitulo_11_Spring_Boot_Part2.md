# Capítulo 11: Spring Boot (Parte 2) - Testes de Integração End-to-End (E2E)

Este capítulo avança para os testes de integração completos (E2E - End to End). O objetivo aqui não é simular (mockar), mas sim subir toda a aplicação — Web Layer, Service Layer, Repositories — integrando as regras de negócio a um servidor web de verdade e lendo e gravando informações em um banco de dados real (como o H2 Database In-Memory). 

## 1. Subindo a Aplicação com `@SpringBootTest`
A anotação principal para subir todo o Application Context do Spring e injetar os Beans verdadeiros é a `@SpringBootTest`.

### Configurando a Porta do Servidor
Dentro dessa anotação, você configura o parâmetro `webEnvironment`:
*   **`WebEnvironment.MOCK` (Padrão):** O servidor web "real" não é iniciado. Os testes rodam em um contêiner web falso e exigem o uso do `MockMvc`.
*   **`WebEnvironment.DEFINED_PORT`:** Inicia um servidor Tomcat real escutando na porta exata definida no arquivo `application.properties` (ex: 8080).
*   **`WebEnvironment.RANDOM_PORT`:** Inicia o Tomcat numa porta livre aleatória. **(Esta é a configuração recomendada para testes)**. Isso evita conflitos (o famoso erro *Port Already In Use*) e permite que múltiplos ambientes de teste rodem em paralelo.
    *   *Dica de sintaxe:* Se precisar descobrir dinamicamente qual porta aleatória foi escolhida, anote uma variável com `@LocalServerPort int porta;`.

## 2. Sobrescrevendo as Configurações de Banco e Servidor
Para não bagunçar as configurações de produção da sua API (que apontam para um MySQL na nuvem, por exemplo), podemos usar um properties dedicado para os testes que rode apenas um H2 local:
Adicione a anotação na classe indicando qual arquivo carregar:
```java
@TestPropertySource(locations = "/application-test.properties")
```

## 3. Realizando Requisições Reais com `TestRestTemplate`
Como o nosso servidor está rodando numa porta aleatória verdadeira (`RANDOM_PORT`), o objeto falso `MockMvc` não funciona mais. O Spring disponibiliza um Client HTTP oficial e pronto para testes de integração: o **`TestRestTemplate`**.

```java
@Autowired
private TestRestTemplate restTemplate;

@Test
void tentarCriarUsuarioViaRequisicaoHTTP() {
    // 1. Arrange: Montar Payload e Headers
    HttpHeaders headers = new HttpHeaders();
    headers.setAccept(Arrays.asList(MediaType.APPLICATION_JSON));
    
    // O HttpEntity engloba os Headers + o Corpo do POST (JSON)
    HttpEntity<UserDetailsRequestModel> entity = new HttpEntity<>(payloadDoUser, headers);

    // 2. Act: Disparar requisição POST real. O parâmetro UserRest.class instrui 
    // o Spring a pegar a resposta JSON e convertê-la magicamente para a classe UserRest.
    ResponseEntity<UserRest> response = restTemplate.postForEntity(
            "/users", 
            entity, 
            UserRest.class
    );

    // 3. Assert: Conferir código HTTP
    assertEquals(HttpStatus.OK, response.getStatusCode());
}
```

## 4. O Fluxo de Testes, Spring Security e Tokens JWT
Em testes End-to-End, dependemos muito de estado, ou seja, testar rotas de Login e Leitura depende de primeiro termos cadastrado um usuário no banco (que no caso do H2, está vazio ao iniciar os testes).
Na aula, organizamos a suíte de testes com a seguinte ordem rigorosa de execução:

1.  **Cria o usuário:** Envia o POST e os dados são salvos fisicamente no H2.
2.  **Tenta acessar dados protegidos (sem token):** Valida se o Spring Security bloqueia e responde com `403 Forbidden`.
3.  **Realiza Login:** Bate no endpoint e recebe sucesso, capturando o **Token JWT** no Header da Resposta (`Authorization`).
4.  **Acessa dados protegidos (com token):** Anexa o Token coletado no Request Header e acessa a rota.

### Compartilhando o Token JWT entre os Testes (Estado)
Há um grande problema técnico ao orquestrar a execução passo-a-passo: **O JUnit 5 cria uma instância nova e zera todas as variáveis da classe para cada método `@Test` que executa**.
Se você logar e colocar o JWT numa variável na etapa 3, quando a etapa 4 iniciar, a variável estará `null`.

Para consertar isso, precisamos alterar o ciclo de vida da classe de testes para **`PER_CLASS`** (Uma instância para a classe inteira), além de ativar a anotação para forçar a ordem de disparo com o `@Order()`.

```java
// O estado das propriedades é mantido para toda a classe!
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class FullIntegrationTest {

    private String authorizationToken; // Vai sobreviver entre um teste e outro

    @Test
    @Order(1)
    void step1_Criacao() { ... }

    @Test
    @Order(2)
    void step2_FazLoginEColetaToken() {
        // ... (envia o login)...
        
        // Coleta o token do Response e salva na classe
        this.authorizationToken = response.getHeaders()
                                          .getValuesAsList("Authorization").get(0); 
    }

    @Test
    @Order(3)
    void step3_AcessaAAPIUsandoTokenColetado() {
        HttpHeaders headers = new HttpHeaders();
        // Usando o token no header Bearer!
        headers.setBearerAuth(authorizationToken); 
        
        // O método exchange permite enviar Headers customizados no TestRestTemplate
        ResponseEntity<UserRest[]> response = restTemplate.exchange(
                "/users", 
                HttpMethod.GET, 
                new HttpEntity<>(headers), 
                UserRest[].class
        );
        
        assertEquals(HttpStatus.OK, response.getStatusCode());
    }
}
```
