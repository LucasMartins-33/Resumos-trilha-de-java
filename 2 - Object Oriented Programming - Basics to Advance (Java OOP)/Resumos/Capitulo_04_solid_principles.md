# Anotações de Estudo: Java Core - Capítulo 4 (SOLID Principles)

> [!NOTE]
> Os princípios SOLID foram popularizados por Robert C. Martin (Uncle Bob) e são a base de um bom design orientado a objetos. O objetivo deles é tornar o código mais **flexível, fácil de entender, sustentável e escalável**.

SOLID é um acrônimo para cinco princípios de design. Vamos mergulhar em cada um deles com os cenários usados no seu curso!

---

## 1. S - Single Responsibility Principle (SRP)
**Princípio da Responsabilidade Única**

* **A Regra**: "Uma classe deve ter apenas um motivo para mudar."
* **A Explicação**: Uma classe deve focar em fazer uma única coisa (ter apenas uma responsabilidade). O tamanho ideal de uma classe é aquele onde você consegue descrever seu propósito em apenas uma frase simples.
* **O Problema no Curso**: Havia uma classe `MailboxSettingsService` que lidava com configurações de e-mail E também tinha um método `hasAccess()` para validar as permissões de segurança do usuário. Isso significa que a classe precisaria ser modificada tanto por mudanças de regras de e-mail quanto por regras de segurança.
* **A Solução**: Criar uma classe separada chamada `SecurityService` encarregada apenas da segurança. O serviço de e-mail agora apenas consome o serviço de segurança. Duas classes, duas responsabilidades bem definidas!

---

## 2. O - Open / Closed Principle (OCP)
**Princípio do Aberto / Fechado**

* **A Regra**: "Entidades de software (classes, módulos) devem ser **abertas para extensão**, mas **fechadas para modificação**."
* **A Explicação**: Você deve ser capaz de adicionar novos comportamentos (extensão) sem precisar alterar o código já escrito e testado (modificação). Como fazer isso? Através do uso de **Abstrações** (Interfaces).
* **O Problema no Curso**: Uma classe `LoanHandler` tinha os métodos `approvePersonalLoan()` e `approveMortgage()`. Se o banco decidisse aprovar financiamento de carro, o desenvolvedor teria que abrir essa classe, modificá-la e correr o risco de quebrar os empréstimos antigos.
* **A Solução**: Cria-se uma interface `LoanHandler` com um método genérico `approveLoan(User)`. A partir daí, criamos `MortgageLoanHandler` e `PersonalLoanHandler` implementando a interface. Se chegar o financiamento de carro, basta criar uma classe totalmente nova `CarLoanHandler`. Extensão realizada com sucesso, sem tocar em código antigo!

---

## 3. L - Liskov Substitution Principle (LSP)
**Princípio da Substituição de Liskov**

* **A Regra**: "Objetos de um tipo devem poder ser substituídos por instâncias de seus subtipos sem alterar a corretude do programa."
* **A Explicação**: Se uma classe herda de outra, você deve conseguir usá-la em qualquer lugar onde a classe pai for exigida, e o código não pode quebrar. Se um filho se recusa a fazer algo que o pai faz, o princípio foi violado.
* **O Problema no Curso**: Um Avestruz (`Ostrich`) herda da interface Pássaro (`Bird`). A interface `Bird` diz que todo pássaro tem que ter o método `fly()` (voar). Mas o Avestruz não voa! Se o desenvolvedor colocar `throw new UnsupportedOperationException()` dentro do método de voo do Avestruz, o programa quebrará em tempo de execução ao tentar iterar por uma lista de Pássaros genéricos mandando todos voarem.
* **A Solução**: Mover o método `fly()` para uma interface segregada chamada `FlyingBird`. A interface `Bird` raiz passa a ter apenas o método `eat()` (comer). Agora o Avestruz só implementa `Bird`, e o programa não espera mais que ele voe. 

---

## 4. I - Interface Segregation Principle (ISP)
**Princípio da Segregação de Interfaces**

* **A Regra**: "Os clientes não devem ser forçados a depender de métodos que não utilizam."
* **A Explicação**: Evite criar interfaces "gordas" (Fat Interfaces) que fazem mil coisas diferentes. É preferível criar várias interfaces "magras" e específicas. Um objeto em Java pode implementar múltiplas interfaces, então aproveite isso.
* **O Problema no Curso**: Uma única interface `Vehicle` declarava os métodos `drive()`, `fly()` e `sail()` (Dirigir, voar e navegar). O problema? Um carro que implementasse `Vehicle` seria forçado a ter métodos de voar e navegar completamente vazios ou inúteis.
* **A Solução**: Quebrar a interface "gorda" em três interfaces finas: `Drivable`, `Flyable`, e `Sailable`. O barco (`Boat`) implementa apenas `Sailable`. Um carro voador pode implementar `Drivable` e `Flyable`. 

---

## 5. D - Dependency Inversion Principle (DIP)
**Princípio da Inversão de Dependência**

* **A Regra**: 
  1. Módulos de alto nível não devem depender de módulos de baixo nível. Ambos devem depender de abstrações. 
  2. Abstrações não devem depender de detalhes. Detalhes devem depender de abstrações.
* **A Explicação**: O coração do seu sistema não deve ficar amarrado às implementações de detalhes (como bancos de dados ou APIs de terceiros). A ligação entre as partes do sistema deve ser feita por meio de **Interfaces**.
* **O Problema no Curso**: A classe central `WeatherAggregator` calculava a temperatura média chamando classes de APIs diretamente: `AccuweatherApi` e `BbcWeatherApi`. Cada API devolvia a temperatura de um jeito (Celsius e Fahrenheit). A classe central precisava conhecer esses detalhes de baixo nível e fazer as conversões internamente.
* **A Solução**: Criou-se a interface de abstração `WeatherSource` com o contrato `getTemperatureCelsius()`. O Agregador de Clima agora não liga mais para a BBC ou Accuweather, ele apenas depende da interface `WeatherSource`. Coube às APIs de baixo nível implementarem a conversão interna para entregar em Celsius. A dependência foi invertida!
