# Anotações de Estudo: Java Core - Capítulo 9 (Clean Architecture)

> [!NOTE]
> Este capítulo é essencial para arquitetos de software e desenvolvedores Sênior. Ele expande os princípios do SOLID para o nível de arquitetura de projeto, ensinando como estruturar pacotes, módulos e a mentalidade por trás de um código que sobrevive a décadas de evolução.

---

## 1. Coupling & Cohesion (Acoplamento e Coesão)

A regra de ouro da arquitetura limpa em uma frase: **Low Coupling & High Cohesion** (Baixo Acoplamento e Alta Coesão).

* **Coupling (Acoplamento)**: É o grau de interdependência entre os módulos. 
  * *Alto Acoplamento* (Ruim): Modificar a classe A te obriga a modificar a classe B (Ripple Effect).
  * *Baixo Acoplamento* (Bom): Módulos são independentes e comunicam-se via interfaces abstratas.
* **Cohesion (Coesão)**: É o grau em que os elementos de uma classe/módulo pertencem juntos.
  * *Baixa Coesão* (Ruim): Uma classe que faz conexões de banco de dados, gera relatórios e envia e-mails (Classe Faz-tudo).
  * *Alta Coesão* (Bom): Uma classe com um propósito único e focado.
* **Plugin Concept**: Na Clean Architecture, a sua *Lógica de Negócios* é o núcleo sagrado. Frameworks, Banco de Dados (ex: Hibernate) e Interface de Usuário são apenas **Plugins** periféricos. Se o banco mudar de SQL para NoSQL, a regra de negócio não deve ser tocada.

---

## 2. Tell, Don't Ask & Anti-simetria de Dados

* **Tell, Don't Ask Principle**: Não peça os dados do objeto para tomar decisões no lugar dele. Diga ao objeto o que ele deve fazer.
  * *Errado*: `if(account.getBalance() > 100) { account.setBalance(account.getBalance() - 100); }`
  * *Correto*: `account.withdraw(100);`
* **Anemic Domain Model (Modelo Anêmico)**: Classes que possuem apenas atributos, Getters e Setters, mas nenhuma regra de negócio. Elas não são objetos de verdade, são apenas "sacos de dados".
* **Object vs Data Structure Anti-symmetry**:
  * **Objetos**: Escondem seus dados (estado privado) e expõem comportamentos (métodos). É fácil criar novas classes de objetos, mas é difícil adicionar novos métodos em todos eles.
  * **Estruturas de Dados (ex: DTOs)**: Expõem dados publicamente e não têm métodos. É fácil adicionar novas funções/serviços que manipulam esses dados, mas é difícil alterar a estrutura de dados em si.
  * *Dica*: Não crie híbridos! Escolha se você precisa de um Objeto rico em comportamento ou de uma simples estrutura de dados para trafegar informações.

---

## 3. Law of Demeter (Lei de Demeter)

Também conhecida como **Princípio do Menor Conhecimento** ("Não fale com estranhos").
* Um método `M` de um objeto `O` só deve chamar métodos de:
  1. Do próprio `O`.
  2. De objetos passados como parâmetros para `M`.
  3. De objetos criados dentro de `M`.
* **Sinal de Violação**: Cadeias de chamadas longas. Ex: `car.getOwner().getWallet().payFine()`. Um Carro não deveria ter acesso direto ao método de "pagar multa" da carteira do dono.
* **Exceções**: A lei não se aplica ao *Builder Pattern* (ex: `car.setDoors(4).setEngine(V8).build()`), pois cada chamada retorna o próprio objeto. Também não se aplica a Estruturas de Dados puras.

---

## 4. KISS, YAGNI e DRY

Princípios clássicos de engenharia para evitar o *Overengineering* (complexidade desnecessária):

* **KISS (Keep It Simple, Stupid)**: Foca na simplicidade do **design do código**. Use lógicas simples, evite padrões de projeto excessivamente complexos para resolver problemas triviais. Códigos "espertinhos" são difíceis de manter.
* **YAGNI (You Aren't Gonna Need It)**: Foca no escopo das **features**. Não implemente nada hoje achando que "talvez possamos precisar disso no futuro". Construa apenas o necessário para os requisitos atuais (MVP).
* **DRY (Don't Repeat Yourself)**: Não repita lógica ou conhecimento. Se a mesma regra de validação de e-mail está em 3 métodos diferentes, encapsule-a em um só lugar.
  * *Antíteses do DRY*: **WET** (Write Everything Twice) e **AHA** (Avoid Hasty Abstractions - Evite criar abstrações mal pensadas e complexas só para fugir da repetição de duas linhas de código. Equilíbrio é tudo).

---

## 5. Principles of Packaging (Como agrupar seu código?)

Ao construir sistemas reais, você dividirá seu código em Pacotes/Módulos. Uncle Bob dita 6 princípios para isso:

### Princípios de Coesão (O que vai dentro do pacote?)
1. **Common Closure Principle (CCP)**: O "Single Responsibility" dos pacotes. Classes que mudam pelos mesmos motivos devem estar no mesmo pacote.
2. **Common Reuse Principle (CRP)**: Classes usadas em conjunto vão no mesmo pacote. Se o cliente depender do seu pacote, ele deve usar *todas* as classes dele, e não apenas uma classe avulsa.
3. **Reuse-Release Equivalence (REP)**: A unidade de reuso é a unidade de release (lançamento de versão).

### Princípios de Acoplamento (Como os pacotes interagem?)
1. **Acyclic Dependencies Principle (ADP)**: Jamais crie ciclos de dependências (Pacote A depende do B, que depende do A). Quebre o ciclo usando a Inversão de Dependência (Crie interfaces).
2. **Stable Dependencies Principle (SDP)**: As dependências devem sempre apontar para a direção da **estabilidade**. Um módulo flexível/volátil (difícil de manter se muita gente depender dele) NUNCA deve ser a fundação de um módulo estável.
3. **Stable Abstractions Principle (SAP)**: Quanto mais estável for um módulo, mais **abstrato** (interfaces) ele deve ser. Isso permite que ele seja estendido futuramente sem modificações. Módulos voláteis devem ser concretos.

### Package by Layer vs Package by Feature
* **Package by Layer (Camadas)**: O padrão MVC de muitos tutoriais (`controllers/`, `services/`, `models/`). *Desvantagem*: Baixa coesão, você edita 5 pacotes diferentes para criar uma simples feature.
* **Package by Feature (Funcionalidades)**: Agrupa pelo domínio do negócio (`orders/`, `billing/`, `users/`). *Vantagem*: Alta coesão, se quiser deletar a funcionalidade, basta apagar uma pasta.
* **Lei de Conway**: Os sistemas arquiteturais tendem a ser reflexos da estrutura de comunicação da empresa que os construiu (times isolados criam camadas isoladas; times ágeis multidisciplinares criam arquiteturas baseadas em features separadas).
