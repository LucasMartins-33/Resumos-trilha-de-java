# Anotações de Estudo: Java Core - Capítulo 10 (GRASP Principles)

> [!NOTE]
> **GRASP** significa *General Responsibility Assignment Software Patterns*. Enquanto o SOLID nos ajuda a refinar a estrutura de uma classe (nível de codificação), o GRASP foca em decisões arquiteturais mais abstratas: **"Quem deve fazer o quê, e por quê?"**. 

---

## 1. Information Expert (Especialista na Informação)
* **O Princípio**: Atribua a responsabilidade à classe que possui a informação (os dados) necessária para cumpri-la.
* **Exemplo**: Se você precisa calcular o total de um item de pedido (`OrderLine`), a própria classe `OrderLine` deve ter o método `calculateTotal()`, pois ela já conhece o Produto e a Quantidade.
* **Anti-patterns**: 
  * *Fake Experts*: Classes utilitárias (`OrderCalculator`) que não possuem dados próprios, mas sugam dados de outras classes para fazer cálculos.
  * *DTOs com Lógica*: Colocar regras de negócio em objetos de transferência de dados (DTOs), que deveriam ser apenas "sacos de dados".

## 2. Creator (Criador)
* **O Princípio**: Quem deve instanciar (criar) um objeto B? A classe A deve criar B se:
  1. A contém ou agrega B (ex: `Order` contém `OrderLine`).
  2. A usa B muito de perto.
  3. A possui os dados necessários para inicializar B (ligação direta com o *Information Expert*).
* **Exceções**: Quando o processo de criação for muito complexo (envolver múltiplos passos, acesso a banco, serviços externos), **não** force a entidade de domínio a criar. Use os padrões de **Factory** ou **Builder**.

## 3. Controller (Controlador)
* **O Princípio**: Atribua a responsabilidade de lidar com eventos do sistema (requisições de UI ou APIs) a uma classe controladora. O Controlador não executa a lógica de negócios; ele a **delega** e orquestra a execução.
* **Tipos**:
  * *Facade Controller*: Representa todo um subsistema (ex: `BankingSystemController`). Bom para sistemas menores.
  * *Use-case Controller*: Um controlador para cada caso de uso específico (ex: `CreateOrderController`). Melhor para escalabilidade.
* **O Perigo**: *Bloated Controllers* (Controladores Inchaços). Evite colocar validação, persistência e envio de e-mails dentro do Controller. Ele deve ser magro (*thin*).

## 4. Low Coupling (Baixo Acoplamento)
* **O Princípio**: Mantenha as classes o menos dependentes possível umas das outras.
* **Técnicas**: Programe voltado a interfaces (abstrações), use Injeção de Dependências.
* **Design Patterns que ajudam**: 
  * *Adapter*: Isola a complexidade e incompatibilidade de sistemas externos.
  * *Facade*: Simplifica o acesso a um subsistema complexo por meio de um único ponto de entrada.

## 5. High Cohesion (Alta Coesão)
* **O Princípio**: Mantenha as classes focadas e com propósitos únicos. Os métodos devem operar sobre os mesmos dados e compartilhar do mesmo objetivo.
* **Métrica**: *LCOM* (Lack of Cohesion in Methods). Se métodos de uma classe não compartilham atributos entre si, a classe provavelmente tem baixa coesão.
* **Anti-patterns**:
  * *Swiss Army Knife (Canivete Suíço)*: Classes "faz-tudo" que calculam, salvam no banco, enviam e-mail e geram relatórios.
  * *Utility Dump*: Classes `Utils` cheias de métodos estáticos sem relação semântica (gerador de senha junto com formatador de data).

## 6. Polymorphism (Polimorfismo)
* **O Princípio**: Quando o comportamento variar de acordo com o tipo ou estado de um objeto, não use instruções condicionais rígidas (longos blocos `if/else` ou `switch`). Atribua o comportamento a diferentes classes utilizando Polimorfismo.
* **Padrões Relacionados**: 
  * *Strategy*: Escolher algoritmos ou comportamentos diferentes (ex: `PaymentStrategy` com implementações `CreditCard`, `PayPal`).
  * *State*: Alterar comportamento quando o estado interno muda.
* **Atenção**: Evite *Overengineering*. Só aplique abstrações polimórficas se houver de fato uma variação de comportamento esperada.

## 7. Pure Fabrication (Fabricação Pura)
* **O Princípio**: Quando colocar uma responsabilidade técnica em um objeto de domínio violaria o *High Cohesion* ou o *Low Coupling*, crie uma classe artificial (que não existe no mundo real/negócio) apenas para essa função.
* **Exemplos**: `Repositories` (lidam com banco de dados), `Services` (coordenam tarefas) e `Managers`.
* **Aviso Crucial**: Cuidado para não cair no anti-pattern de *Anemic Domain Model* (Modelo de Domínio Anêmico), onde você suga todas as regras de negócio para as classes *Service* e deixa as entidades apenas com getters/setters. As entidades de domínio ainda devem ser especialistas em seus próprios dados.

## 8. Indirection (Indireção)
* **O Princípio**: Para evitar o acoplamento direto entre dois ou mais componentes, introduza um intermediário entre eles.
* **Exemplos de Indireção**:
  * *Event Bus*: Produtores de eventos não conhecem os consumidores.
  * *Padrão Mediator*: Objetos não conversam entre si, tudo passa pelo mediador (ex: regras de uma sala de chat).
  * *Controllers*: Separam a UI da camada de Domínio.
