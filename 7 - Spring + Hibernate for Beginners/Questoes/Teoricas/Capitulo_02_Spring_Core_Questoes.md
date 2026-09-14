# Questões Teóricas - Capítulo 02: Spring Core

### Questão 1
O que é o conceito de **Inversão de Controle (IoC - Inversion of Control)** no ecossistema Spring?

<details>
<summary>👀 Ver Resposta</summary>

Inversão de Controle é um princípio de design onde a criação, gerenciamento e controle do ciclo de vida dos objetos (Beans) é terceirizada para o container do framework (Spring IoC Container), em vez de ser gerenciada manualmente pelo desenvolvedor usando o operador `new`.
</details>

---

### Questão 2
Explique o funcionamento e as diferenças entre os três principais tipos de Injeção de Dependência no Spring: Constructor, Setter e Field Injection.

<details>
<summary>👀 Ver Resposta</summary>

* **Constructor Injection (Recomendado)**: As dependências são passadas através do construtor da classe. Garante imutabilidade (`final`), impede dependências nulas e facilita testes unitários.
* **Setter Injection**: As dependências são injetadas através de métodos setter. Útil para dependências opcionais ou dinâmicas.
* **Field Injection**: Dependências injetadas diretamente nos atributos via `@Autowired`. Desencorajado pela comunidade por dificultar testes sem container e ocultar acoplamentos excessivos.
</details>

---

### Questão 3
Quando ocorre a exceção `NoUniqueBeanDefinitionException` e como as anotações `@Qualifier` e `@Primary` resolvem esse problema?

<details>
<summary>👀 Ver Resposta</summary>

Essa exceção ocorre quando o Spring encontra mais de um Bean que implementa a mesma interface solicitada para injeção.
* **`@Primary`**: Define um Bean como a escolha prioritária padrão caso nenhum qualificador seja especificado.
* **`@Qualifier("nomeDoBean")`**: Define explicitamente no ponto de injeção o nome exato do Bean que deve ser injetado, possuindo maior prioridade que o `@Primary`.
</details>

---

### Questão 4
Diferencie os escopos de Bean `singleton` e `prototype` no Spring Framework.

<details>
<summary>👀 Ver Resposta</summary>

* **`singleton` (Padrão)**: O container Spring cria apenas uma única instância do Bean para toda a aplicação. Todos os pontos de injeção compartilham a mesma instância.
* **`prototype`**: O container Spring cria uma nova instância do Bean a cada vez que ele é solicitado ou injetado.
</details>

---

### Questão 5
Como funcionam os métodos de ciclo de vida `@PostConstruct` e `@PreDestroy` em um Spring Bean?

<details>
<summary>👀 Ver Resposta</summary>

* **`@PostConstruct`**: Executado imediatamente após o container instanciar o Bean e injetar todas as suas dependências. Usado para inicializações ou conexões com recursos.
* **`@PreDestroy`**: Executado imediatamente antes do Bean ser destruído pelo container (apenas para Beans no escopo Singleton). Usado para limpeza de recursos e encerramento de conexões.
</details>

---

### Questão 6
Qual é o papel da anotação `@Configuration` e da anotação `@Bean` no Java Config do Spring?

<details>
<summary>👀 Ver Resposta</summary>

A anotação `@Configuration` indica que uma classe contém métodos de configuração de Beans do Spring. A anotação `@Bean` é colocada sobre um método dentro dessa classe para declarar que o objeto retornado por esse método deve ser registrado e gerenciado como um Bean pelo IoC Container (útil para expor bibliotecas de terceiros como Beans).
</details>

---

### Questão 7
O que é o **Spring Component Scanning** e como ele identifica automaticamente as classes gerenciadas?

<details>
<summary>👀 Ver Resposta</summary>

É o processo onde o Spring faz a varredura do classpath procurando por classes anotadas com estereótipos como `@Component`, `@Controller`, `@RestController`, `@Service` e `@Repository`. Ao encontrá-las, o Spring registra-as automaticamente como Beans no IoC Container.
</details>

---

### Questão 8
Qual é a diferença conceitual entre as anotações estereótipos `@Component`, `@Service` e `@Repository`?

<details>
<summary>👀 Ver Resposta</summary>

Do ponto de vista do Spring, todas são especializações de `@Component`. Porém:
* **`@Component`**: Utilitários ou componentes genéricos.
* **`@Service`**: Indica que a classe contém regras e lógica de negócio.
* **`@Repository`**: Indica a camada de acesso a dados (DAO/Persistência) e habilita a tradução automática de exceções de banco de dados nativas para exceções Spring (`DataAccessException`).
</details>

---

### Questão 9
Por que Beans com escopo `prototype` exigem cuidado especial com o método `@PreDestroy`?

<details>
<summary>👀 Ver Resposta</summary>

Porque o Spring não gerencia o ciclo de vida completo de Beans no escopo `prototype`. O Spring instancia, configura e injeta o Bean `prototype`, mas passa a responsabilidade de destruição para o código chamador. Portanto, o método `@PreDestroy` **não é chamado automaticamente** para Beans `prototype`.
</details>

---

### Questão 10
Como injetar valores de propriedades de arquivos de configuração (como `application.properties`) diretamente em variáveis Java usando o Spring?

<details>
<summary>👀 Ver Resposta</summary>

Utiliza-se a anotação `@Value("${nome.da.propriedade}")` sobre o campo, parâmetro do construtor ou método setter da classe gerenciada pelo Spring.
</details>
