# Capítulo 06: Objects, Classes & Data Structures

Embora este curso não seja focado exclusivamente em Programação Orientada a Objetos (OOP), quase todo projeto utiliza classes de alguma forma. A escrita de classes limpas impacta diretamente na leitura e manutenção do sistema.

## 1. Objetos Reais vs Containers de Dados
Nunca misture os dois conceitos. Saiba o que você está criando:

- **Objetos Reais (Real Objects):** Eles *escondem* seus dados (estado privado) e expõem uma API pública (métodos). Nós dizemos a eles o que fazer (abstrações) e não nos importamos com *como* eles fazem.
  *Exemplo:* Classe `Database` com métodos `connect()` e `disconnect()`. Não devemos conseguir acessar a `Connection` privada dela.
- **Estruturas de Dados / Containers:** Apenas armazenam e transportam dados. Não possuem lógica de negócios ou métodos complexos. Suas propriedades são públicas. 
  *Exemplo:* Um objeto de payload de API (ex: `UserCredentials { email, password }`).

Misturar os dois (ex: um Objeto Real que expõe propriedades de conexão permitindo que alguém fora dele chame `.close()`) gera acoplamento, espalha regras de negócio pelo código e fere o Clean Code.

---

## 2. Polimorfismo em Classes
Se você possui métodos lotados de blocos `switch/case` ou `if/else` checando o "tipo" do objeto (ex: `if (deliveryType === 'EXPRESS') ... else if (deliveryType === 'STANDARD') ...`), utilize **Polimorfismo**.

Em vez de uma única classe "Gorda" de Delivery, crie:
1. Uma classe/interface base `Delivery` com um método genérico `deliverProduct()`.
2. Classes especializadas que herdam dessa base (`ExpressDelivery`, `StandardDelivery`), cada uma implementando sua lógica limpa dentro do seu próprio `deliverProduct()`, sem nenhum `if`.
3. Quem chama o método não precisa saber *qual* entrega é, apenas executa `delivery.deliverProduct()`.

---

## 3. Classes devem ser Pequenas e o Princípio SRP
Assim como funções, classes precisam ser pequenas. Porém, a métrica muda. Funções devem "Fazer Apenas Uma Coisa", enquanto **Classes devem ter Apenas Uma Responsabilidade (Single Responsibility Principle - SRP)**.

Uma responsabilidade é um grupo lógico ou uma "razão para mudar". 
Criar, atualizar e deletar um Produto não são "3 coisas separadas que exigem 3 classes". Eles pertencem à **única responsabilidade** de "Gerenciamento de Produtos". Por isso, podem habitar a mesma classe pequena. Mas, se a mesma classe tenta gerar um relatório financeiro, ela assumiu uma responsabilidade nova (e deve ser quebrada).

### Coesão (Cohesion)
A coesão dita o quão unida a sua classe é. 
Uma classe altamente coesa é aquela em que **a maioria dos métodos utiliza a maioria das propriedades**. Se você tem uma classe onde 4 métodos usam apenas a propriedade `A` e outros 4 métodos usam apenas a propriedade `B`, essa classe tem baixa coesão e está claramente implorando para ser dividida em duas classes menores.

---

## 4. A Lei de Demeter (Tell, Don't Ask)
A Lei de Demeter (Princípio do Menor Conhecimento) foca no baixo acoplamento e na legibilidade. Ela dita que você **não deve acessar estruturas profundas de objetos alheios**.

```javascript
// 🔴 Ruim (Violando a lei com Train Wrecks / Encadeamento)
const date = customer.lastPurchase.date.getMonth();

// 🟢 Clean (Tell, Don't Ask - Peça para o objeto realizar a ação)
// Não pergunte os dados ao cliente para você processar, DÊ a instrução a ele:
warehouse.deliver(customer.lastPurchase); 
```
Você só deve interagir com as propriedades imediatas da sua própria classe, objetos criados por você mesmo no escopo, ou argumentos passados diretamente ao método.

---

## 5. Os Princípios S.O.L.I.D.
Um pilar da Orientação a Objetos que ajuda a escrever código limpo e extensível:

- **[S] Single Responsibility:** Uma classe deve ter apenas uma responsabilidade (uma razão para mudar). Evita as famosas "God Classes" (Classes Deus que fazem tudo).
- **[O] Open-Closed Principle:** Classes devem estar **abertas para extensão, mas fechadas para modificação**. Se para adicionar uma nova funcionalidade (ex: Imprimir Excel) você precisa alterar uma classe antiga adicionando um novo `if`, você quebrou o princípio. Crie uma classe nova que estende/implementa uma interface existente!
- **[L] Liskov Substitution:** Você deve conseguir substituir uma classe Base (Pai) por uma de suas filhas sem quebrar nada. (Ex: Um `Penguin` não deveria estender `Bird` se `Bird` tem um método `fly()`, pois pinguins não voam e forçariam a reescrita torta do método).
- **[I] Interface Segregation:** Muitas interfaces pequenas, específicas e focadas em quem as consome são melhores que uma "Interface Geral e gorda". Não force classes a implementarem métodos que elas não precisam (ex: forçar um banco local a implementar `connect()`).
- **[D] Dependency Inversion:** Módulos de alto nível não devem depender de detalhes de baixo nível (implementações concretas). Ambos devem depender de **Abstrações** (Interfaces). Você não cria o banco dentro do método, você injeta o banco (que respeita a interface `Database`) pelo construtor.
