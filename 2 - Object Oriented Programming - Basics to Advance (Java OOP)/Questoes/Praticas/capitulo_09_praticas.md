📘 Capítulo 9: Clean Architecture (Arquitetura Limpa)

**O Cenário:**
Você herdou um projeto onde o Banco de Dados e as Regras de Negócio estão totalmente misturados no `Controller` principal.

Sua missão é simular a quebra desse monolito acoplado usando princípios da Clean Architecture. (Você pode usar apenas classes locais e interfaces para simular as camadas).

🟢 Atividade 9.1: A Entidade Pura (Core)
1. Crie o pacote (ou comente `// Camada de Entidades`).
2. Crie a classe `Usuario`. Ela não deve importar NENHUMA biblioteca externa, nem mesmo anotações do Spring ou Hibernate.
3. Adicione validações de negócio nela (ex: método `bloquearSeInadimplente()`).

🟢 Atividade 9.2: O Caso de Uso (Use Case)
1. Crie a camada de `UseCases`.
2. Crie a classe `RegistrarUsuarioUseCase`.
3. Incorpore nela a lógica de orquestração: Validar dados, criar a entidade `Usuario` e salvar.

🟡 Atividade 9.3: Inversão de Controle no Banco de Dados
Para salvar, o Use Case não pode conhecer o banco (SQL).
1. Crie uma interface `UsuarioRepository` dentro da camada do Use Case (ou Core).
2. O método `RegistrarUsuarioUseCase` deve chamar `repository.salvar(usuario)`.

🟡 Atividade 9.4: O Adaptador de Interface (Interface Adapter)
1. Crie a camada de `Adapters`.
2. Crie a classe `UsuarioController` (fingindo ser um endpoint REST).
3. Faça o Controller receber strings, converter para um formato interno (DTO) e chamar o `RegistrarUsuarioUseCase`.

🟠 Atividade 9.5: Implementando o Detalhe Explicito (Gateway)
1. Crie a camada de `Infra` (Detalhes).
2. Crie a classe `PostgresUsuarioRepository` que implementa a interface `UsuarioRepository`.
3. Adicione `System.out.println("Salvando no Postgres");` para simular o banco.

🟠 Atividade 9.6: A Regra da Dependência (Dependency Rule)
1. Revise seus `imports`.
2. Valide (visualmente ou por comentários) que o pacote `Entities` não importa nada.
3. Valide que `UseCases` importa apenas `Entities`.
4. Valide que `Infra` importa as interfaces de `UseCases`. A seta sempre aponta para o centro!

🔴 Atividade 9.7: Protegendo as Entidades com DTOs
1. O `Controller` não deve retornar a Entidade `Usuario` bruta para a tela.
2. Crie um `UsuarioResponseDTO`.
3. Mapeie a entidade para o DTO no Controller antes de dar o retorno ao chamador.

🔴 Atividade 9.8: Boundary Interfaces (Input/Output Ports)
Em arquiteturas estritas, o UseCase não devolve valores diretamente, ele usa "Ports".
1. Crie uma interface `UsuarioOutputPort` com o método `apresentarResultado()`.
2. Altere o Use Case para chamar essa porta ao finalizar o registro.

🔴 Atividade 9.9: O Presenter
1. Crie um `UsuarioPresenter` implementando a porta de saída criada acima.
2. Faça ele formatar os dados de sucesso para um formato visual.

🔴 Atividade 9.10: Montando tudo (Dependency Injection Root)
1. Crie a classe `Main` (que atua como framework ou DI container).
2. Instancie o Repositorio (Postgres), passe-o para o UseCase.
3. Instancie o UseCase, passe-o para o Controller.
4. Execute e veja o fluxo limpo de fora para dentro.
