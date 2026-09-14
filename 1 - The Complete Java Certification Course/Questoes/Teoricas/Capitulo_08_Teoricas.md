# Questões Teóricas - Capítulo 08 (Understanding Object Orientation)

**1. Classe vs. Objeto:** Qual a diferença conceitual e de tempo de vida entre uma "Classe" e um "Objeto" no Java?
> [!faq]- 👀 Ver Resposta
> Uma classe é apenas um modelo/planta (blueprint) escrito em código (design-time), que define quais atributos e métodos existirão. Ela não faz nada por si só. Um objeto é a manifestação física desse modelo na memória do computador, existindo apenas enquanto o programa está rodando (runtime).

**2. A Regra do Construtor:** O que é um Construtor no Java e quais são as duas regras básicas de sintaxe que o diferenciam de um método comum?
> [!faq]- 👀 Ver Resposta
> O construtor é uma estrutura especial chamada no momento em que usamos o 
ew` para construir o objeto na memória, permitindo inicializar os atributos logo de cara. As duas regras são: ele **precisa ter exatamente o mesmo nome da classe**, e **não possui tipo de retorno** (nem mesmo `void`).

**3. O uso do 'this':** Para que serve a palavra-chave `this` quando usada dentro de um construtor ou método de instância (ex: `this.nome = nome;`)?
> [!faq]- 👀 Ver Resposta
> A palavra `this` faz o objeto referenciar a si mesmo. É usada principalmente para resolver ambiguidades (shadowing), indicando ao Java que `this.nome` se refere à variável (atributo) permanente do objeto, e não ao argumento de mesmo nome passado nos parênteses do método/construtor.

**4. Gerenciamento de Memória (Stack vs Heap):** Explique resumidamente como o Java divide a memória Stack e a memória Heap na criação e uso de variáveis e objetos.
> [!faq]- 👀 Ver Resposta
> A **Stack (Pilha)** é rápida e organizada, e guarda a execução dos métodos e as variáveis locais (inclusive as "variáveis de referência" de objetos). Já a **Heap** é a área vasta onde os objetos criados com o 
ew` (dados complexos) efetivamente "moram". A variável na Stack apenas aponta (referencia) o endereço do objeto morando na Heap.

**5. O Faxineiro da Memória:** Qual é o papel do *Garbage Collector* (Coletor de Lixo) dentro da memória Heap do Java?
> [!faq]- 👀 Ver Resposta
> É um processo automático da JVM que monitora a memória Heap. Quando um objeto lá não tem mais nenhuma "variável de referência" na Stack apontando para ele, ele é considerado "órfão" (inatingível). O Garbage Collector destrói esse objeto e libera a memória automaticamente, prevenindo vazamentos de memória (Memory Leaks).

**6. Herança (Inheritance):** O que é herança e como ela é declarada no código? O Java permite que uma classe herde de várias classes ao mesmo tempo (Herança Múltipla)?
> [!faq]- 👀 Ver Resposta
> A herança (criada com a palavra `extends`) permite que uma classe filha herde automaticamente atributos e métodos de uma classe pai, evitando código duplicado. O Java **não permite herança múltipla de classes**; uma classe só pode ter UM único pai direto (embora possa herdar de um avô, bisavô etc.).

**7. Invocando os Pais com 'super':** Se uma classe `Pai` tem um construtor que exige um argumento `int idade`, qual a obrigação da classe `Filha` e qual palavra-chave é usada?
> [!faq]- 👀 Ver Resposta
> A classe Filha é obrigada a criar seu próprio construtor e usar a instrução `super(idade)` na primeira linha para repassar o argumento e inicializar o estado da classe Pai antes da Filha.

**8. Contratos de Código (Interfaces):** O que são Interfaces no Java e o que a classe é obrigada a fazer quando usa a palavra-chave `implements`?
> [!faq]- 👀 Ver Resposta
> Interfaces são "contratos" puramente abstratos que definem APENAS as assinaturas dos métodos (o que o objeto deve saber fazer), mas sem o corpo/lógica (como fazer). Uma classe que implementa uma Interface assina esse contrato e é **obrigada a sobrescrever/construir** o corpo lógico de todos os métodos definidos nela, senão o código não compila.

**9. Classes Abstratas:** Qual a diferença fundamental entre criar uma `class` tradicional e uma `abstract class`? E em que cenário você prefere usá-la em relação a uma Interface?
> [!faq]- 👀 Ver Resposta
> Uma classe abstrata atua como um molde e **não pode ser instanciada** com o 
ew`. Ao contrário de uma Interface (que é 100% abstrata), a Classe Abstrata permite ter alguns métodos abstratos vazios, MAS também permite ter métodos prontos (com código). Escolhe-se usá-la quando diversas classes filhas precisam herdar um código pronto em comum, mas ainda exigem que comportamentos específicos sejam forçados pela abstração.

**10. O Superpoder do Polimorfismo:** Explique a frase: "A Variável de Referência define o que podemos ver; O Objeto instanciado define como ele se comportará" no contexto do Polimorfismo em Java.
> [!faq]- 👀 Ver Resposta
> Se declararmos `Animal a = new Cachorro();`, o tipo genérico `Animal` (variável) determina que a nossa IDE só vai nos deixar chamar métodos que existam no molde Animal (não podemos chamar um `cavarBuraco` exclusivo do Cachorro). Porém, no momento (runtime) em que chamarmos um método genérico compartilhado como `a.emitirSom()`, o Java olhará para o Objeto de fato (`Cachorro`) e executará o latido programado nele, e não o som genérico do Animal.




