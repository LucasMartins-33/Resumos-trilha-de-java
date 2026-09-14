# Questões Teóricas - Capítulo: Test Driven Development (TDD) em Java

Testes de fixação sobre a metodologia TDD, o ciclo Red-Green-Refactor, design orientado a testes, refatoração de suítes e cenários negativos.

---

### 1. O que é a metodologia Test-Driven Development (TDD) e em que ela difere da abordagem tradicional de desenvolvimento de software?

> [!faq]- 👀 Ver Resposta
> O TDD (Desenvolvimento Orientado a Testes) é uma prática de engenharia de software na qual a escrita dos testes automatizados precede a escrita do próprio código de produção. Na abordagem tradicional, o desenvolvedor implementa toda a lógica de produção e somente depois escreve testes para verificar se o código funciona (ou muitas vezes não escreve). No TDD, o código de produção só é escrito com o propósito explícito de fazer passar um teste previamente reprovado.

---

### 2. Explique detalhadamente as três fases fundamentais do ciclo do TDD: Red, Green e Refactor.

> [!faq]- 👀 Ver Resposta
> * **Red (Vermelho):** Escreve-se um teste unitário que define a próxima regra de negócio desejada. Como a lógica de produção ainda não existe (ou a classe/método sequer foi criada), o teste falha ou nem compila.
> * **Green (Verde):** Implementa-se a quantidade estritamente mínima de código de produção necessária para fazer o teste passar com sucesso.
> * **Refactor (Refatorar):** Com a garantia de segurança do teste passando (barra verde), limpa-se o código de produção e o código de teste, eliminando duplicações, melhorando a clareza e aplicando padrões de projeto, garantindo que tudo permaneça verde.

---

### 3. Por que na fase "Green" do TDD a regra recomenda escrever apenas a quantidade *mínima* de código para satisfazer o teste, mesmo que pareça uma solução ingênua?

> [!faq]- 👀 Ver Resposta
> Porque essa restrição previne a escrita de código especulativo ou desnecessário (princípio YAGNI - *You Aren't Gonna Need It*). Escrever apenas o estritamente suficiente mantém o foco absoluto no requisito atual, evita a introdução prematura de complexidade não testada e garante que todo e qualquer trecho de código adicionado ao projeto tenha uma razão comprovada de existir apoiada por um teste unitário.

---

### 4. O que significa o conceito de "O Teste como o Primeiro Cliente da sua API" no contexto de TDD?

> [!faq]- 👀 Ver Resposta
> Significa que, ao escrever o teste antes de implementar a classe, você assume a perspectiva do consumidor do seu próprio código. Isso força o desenvolvedor a refletir sobre a usabilidade da API: nomes de métodos mais intuitivos, quantidades razoáveis de parâmetros, retornos limpos e facilidade de instanciar e configurar o componente. O teste atua como uma ferramenta de design de software, e não apenas como um validador de bugs.

---

### 5. Como o TDD incentiva a boa prática de "Programar para Interfaces" (Program to an Interface)?

> [!faq]- 👀 Ver Resposta
> Durante a fase Red, quando o teste demanda a criação de um serviço (como `UserService`), a prática de TDD incentiva a criar primeiro a interface com a assinatura necessária e fazer a classe de implementação concreta (`UserServiceImpl`) herdá-la. Isso desacopla o contrato da implementação, facilita a injeção de dependências e viabiliza a criação de dublês de teste (mocks) em componentes que venham a consumir esse serviço posteriormente.

---

### 6. A etapa "Refactor" do TDD aplica-se apenas ao código de produção ou também ao código de testes? Justifique.

> [!faq]- 👀 Ver Resposta
> Aplica-se **a ambos com igual rigor**. O código de teste é um cidadão de primeira classe no repositório do projeto: se for deixado sujo, duplicado e desorganizado, o custo de manutenção da suíte se tornará insustentável. Durante a refatoração, o desenvolvedor deve eliminar repetições na preparação dos dados de teste, extrair métodos auxiliares, adotar anotações de ciclo de vida (`@BeforeEach`) e aprimorar a legibilidade das asserções.

---

### 7. Quando a etapa de Arrange (Preparação) começa a se repetir de forma idêntica em vários métodos de teste da mesma classe, qual é a refatoração indicada?

> [!faq]- 👀 Ver Resposta
> A refatoração indicada é extrair a inicialização comum dos dados e a instanciação do objeto sob teste para um método de configuração anotado com **`@BeforeEach`**. Isso elimina a duplicação entre os métodos de teste, mantém cada método focado apenas no cenário específico de ação (*Act*) e validação (*Assert*), e assegura que cada teste continue recebendo instâncias limpas e isoladas.

---

### 8. Em quais situações é recomendável consolidar ou agrupar asserções que estavam inicialmente distribuídas em testes separados?

> [!faq]- 👀 Ver Resposta
> Quando dois ou mais testes executam o exato mesmo caminho de execução com os mesmos parâmetros de entrada e diferem apenas por testar atributos complementares do mesmo resultado (por exemplo, um teste checando se o primeiro nome foi gravado e outro checando se o sobrenome foi gravado). Agrupá-los em um único teste contendo múltiplas asserções reduz o tempo total de execução da suíte e remove redundâncias desnecessárias sem perda de clareza.

---

### 9. Como se aplica o ciclo Red-Green-Refactor ao desenvolvimento de regras de validação que lançam exceções (cenários negativos)?

> [!faq]- 👀 Ver Resposta
> 1. **Red:** Escreve-se um teste passando dados ilegais (como uma string de nome vazia `""`) e envolve-se a chamada no método `assertThrows(IllegalArgumentException.class, ...)`. Como o código de produção ainda não valida o argumento, o teste falha (vermelho).
> 2. **Green:** No método de produção, insere-se a validação condicional simples: `if (name.trim().isEmpty()) { throw new IllegalArgumentException("Nome inválido"); }`. O teste passa (verde).
> 3. **Refactor:** Com o teste verde garantido, refatora-se a mensagem de erro para uma constante ou classe padronizada de mensagens e otimiza-se o algoritmo de validação.

---

### 10. Qual é a principal vantagem que o TDD oferece para a realização de grandes refatorações no futuro do sistema?

> [!faq]- 👀 Ver Resposta
> O TDD constrói de forma inerente uma rede de segurança de testes unitários abrangente e de alta granularidade desde o dia um do projeto. No futuro, quando a arquitetura, algoritmos internos ou dependências de terceiros precisarem ser refatorados para ganho de performance ou modernização, o desenvolvedor poderá alterar profundamente o código de produção com a confiança de que qualquer quebra de contrato ou regressão funcional será apontada imediatamente pelos testes existentes.
