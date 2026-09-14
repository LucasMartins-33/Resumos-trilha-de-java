# Anotações de Estudo: Java Core - Capítulo 15 (Structural Diagrams)

> [!NOTE]
> Ao contrário dos Diagramas Comportamentais (que focam no tempo, eventos e estados dinâmicos), os **Diagramas Estruturais** modelam a arquitetura estática de um sistema. Eles representam os "blocos de construção" do software: classes, componentes, pacotes e a infraestrutura física de hardware.

---

## 1. Class Diagram (Diagrama de Classes)
É o diagrama estrutural mais importante e utilizado. Serve como um "projeto" (blueprint) estático do sistema, detalhando atributos, métodos e relações de Orientação a Objetos.

### Estrutura Visual da Classe
A classe é um retângulo dividido em três seções:
1. **Nome (Upper)**: No centro, em negrito. Se for uma *Classe Abstrata*, o nome vai em *itálico*.
2. **Atributos (Middle)**: Segue o formato `Visibilidade Nome : Tipo`. Ex: `- email: String`.
3. **Métodos (Lower)**: Segue o formato `Visibilidade Nome(parametros) : TipoRetorno`. Ex: `+ processPayment(amount: double): void`.

### Modificadores de Acesso (Visibilidade)
* `+` : Public
* `-` : Private
* `#` : Protected
* `~` : Package-Private (Default no Java)

### Tipos de Relacionamentos
* **Association (Associação)**: Linha simples. Relacionamento fraco. Ex: `Estudante` estuda `Curso`.
* **Aggregation (Agregação)**: Linha com **losango vazio** (apontando pro Todo). Relacionamento Todo-Parte onde a parte **pode existir** sem o todo. Ex: `Carro` e `Roda`.
* **Composition (Composição)**: Linha com **losango preenchido** (apontando pro Todo). Relacionamento Todo-Parte forte, a parte **morre** se o todo for destruído. Ex: `Casa` e `Quarto`.
* **Dependency (Dependência)**: Linha **tracejada com seta aberta**. Uma classe usa a outra (ex: em parâmetros de método), mas não a possui como atributo fixo.
* **Generalization/Inheritance (Herança)**: Linha **cheia com seta vazia (branca)** apontando para a classe pai (`extends`).
* **Interface Implementation**: Linha **tracejada com seta vazia (branca)** apontando para a interface (`implements`). Interfaces usam a notação `<<interface>>`.

### Multiplicity (Cardinalidade)
* `1` : Exatamente um.
* `0..1` : Zero ou um.
* `0..*` ou `*` : Zero ou muitos.
* `1..*` : Um ou muitos.
* `M..N` : Ex: `2..5` (No mínimo 2, no máximo 5).

---

## 2. Object Diagram (Diagrama de Objetos)
É uma "fotografia" (snapshot) do Diagrama de Classes em um momento específico do tempo (runtime).
* Modela **Instâncias**, não os moldes.
* **Notação de Nome**: `identificador : NomeDaClasse` sublinhado. Ex: `d1 : Department`. Se for anônimo: `: ContactInfo`.
* **Atributos**: Diferente da classe, aqui mostramos os **valores atuais**. Ex: `id = 123`.
* **Uso**: Excelente para visualizar cenários de testes, debugar estados estranhos no sistema em runtime ou entender como objetos se ligam na prática (Links).

---

## 3. Component Diagram (Diagrama de Componentes)
Focado em decompor grandes sistemas em blocos modulares, físicos (DLLs, executáveis) ou lógicos (Módulos de negócio). Eles atuam como caixas-pretas onde só nos importamos com as *Interfaces* que eles fornecem e consomem.

* **Component**: Retângulo com o ícone de componente no canto.
* **Provided Interface (Interface Fornecida)**: Círculo (Lollipop/Ball) ligado ao componente. O serviço que ele oferece.
* **Required Interface (Interface Requerida)**: Semicírculo (Socket/Cup) ligado ao componente. O serviço que ele precisa para funcionar.
* **Assembly Connector**: A junção de um círculo e um semicírculo (Ball and Socket), conectando componentes.
* **Port (Porta)**: Um pequeno quadrado na borda do componente indicando um ponto exato de interação com o ambiente externo.

---

## 4. Package Diagram (Diagrama de Pacotes)
Agrupa elementos (classes, interfaces, componentes) em namespaces (pacotes), como pastas de arquivos. Ajuda na modularidade e organização em arquiteturas gigantescas.

* **Package**: Visual de "pasta de arquivos".
* **Dependências Especiais**:
  * `<<import>>`: Uma dependência onde o pacote Cliente usa os serviços do pacote Fornecedor.
  * `<<access>>`: Similar, mas indica uma necessidade explícita de visibilidade aos elementos privados/protegidos de outro pacote.

---

## 5. Deployment Diagram (Diagrama de Implantação)
Modela a topologia de hardware e como os artefatos de software rodam nesse hardware.
* **Node (Nó)**: Dispositivo físico de hardware (Servidor, Máquina do Cliente, Roteador). Representado por um cubo 3D.
* **Artifact**: O software real compilado implantado (arquivo `.jar`, um banco de dados).
* **Communication Paths**: Linhas conectando os nós, indicando como se comunicam (ex: com um rótulo `TCP/IP` ou `HTTPS`).
* **Manifestation (`<<manifest>>`)**: Relaciona um Componente Lógico ao Artefato Físico que o implementa.

---

## 6. Composite Structure Diagram (Estrutura Composta)
Aprofunda a visão **interna** de um classificador complexo (uma Classe muito grande ou um Sistema inteiro). Ideal para mostrar como os subsistemas conversam lá dentro.
* **Part**: Um retângulo interno que faz parte da estrutura.
* **Role**: Qual o papel daquela parte. Ex: `:MainBoard` (Placa mãe atuando num PC).
* Você desenha os **Ports** e os **Connectors** interligando diretamente as Parts lá dentro. Excelente para modelar hardware interno (ex: Slots de Memória conectando via barramento com a CPU) ou softwares baseados em micro-componentes altamente acoplados internamente.

---

## 7. Profile Diagram (Diagrama de Perfil)
Permite criar a sua própria versão customizada da UML (Domain-Specific). Usado para criar extensões voltadas para frameworks específicos (como Java EE, C#, Aeroespacial).

* **Metaclass**: Uma classe padrão do metamodelo UML (ex: A ideia de ser uma `Class`, `Component`, `Dependency`).
* **Stereotype (`<<stereotype>>`)**: Cria uma nova palavra/rótulo (ex: `<<EJBBean>>`) que estende (Extension - seta preenchida) uma Metaclass.
* **Tagged Values**: Parâmetros de chave-valor atrelados a um estereótipo para guardar metadados. Ex: Um `state` com valor `stateless`.
* **Constraints**: Regras de negócio adicionais muitas vezes escritas em OCL (Object Constraint Language).
