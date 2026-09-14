# Anotações de Estudo: Java Core - Capítulo 13 (UML Introduction)

> [!NOTE]
> A UML (**Unified Modeling Language**) não é uma linguagem de programação, mas uma linguagem visual padronizada para visualizar, especificar, construir e documentar a arquitetura e os artefatos de um sistema de software.

---

## 1. Por que aprender UML?
Entender a UML é uma parte fundamental de aprender Programação Orientada a Objetos porque ela permite:
* **Visualização e Abstração**: Transforma conceitos complexos de OOP (como herança, polimorfismo e associações) em representações visuais claras, ignorando detalhes de código desnecessários na fase de design.
* **Comunicação**: Funciona como um "idioma comum" entre Desenvolvedores, Arquitetos, Analistas de Negócios (BAs), Gerentes de Projeto e *Stakeholders* (clientes).
* **Geração de Código**: Algumas ferramentas avançadas conseguem ler diagramas UML estruturados e gerar os esqueletos das classes automaticamente.
* **Prevenção de Erros**: Planejar a arquitetura na fase de design evita bugs complexos antes mesmo da primeira linha de código ser escrita.

---

## 2. Visão Geral da UML
* **Histórico**: Desenvolvida na Rational Software em 1994 (por Grady Booch, Ivar Jacobson e James Rumbaugh). Adotada como padrão pelo OMG (Object Management Group) em 1997.
* **Versão Atual**: A versão base moderna de mercado é a **UML 2.5**, que simplificou a documentação e notação, mantendo os mesmos 14 tipos de diagramas introduzidos na 2.0.

---

## 3. Os 14 Tipos de Diagramas da UML
A UML divide seus diagramas em **dois grandes grupos**, dependendo se o foco é a estrutura estática (como as coisas são) ou o comportamento dinâmico (como as coisas agem).

### A. Diagramas Estruturais (Structure Diagrams)
Focam nos elementos estáticos e na composição arquitetural do sistema.
1. **Class Diagram (Diagrama de Classes)**: O mais famoso em OOP. Modela classes, atributos, métodos e os relacionamentos (herança, composição, agregação) entre elas.
2. **Object Diagram**: Fotografia do sistema em tempo de execução (estado dos objetos instanciados).
3. **Package Diagram**: Organiza classes em pacotes (namespaces) para lidar com a complexidade.
4. **Component Diagram**: Mostra os componentes físicos do software (ex: bibliotecas, executáveis) e suas dependências.
5. **Composite Structure Diagram**: Foca na estrutura interna de classificadores complexos usando portas e conectores.
6. **Deployment Diagram (Diagrama de Implantação)**: Modela a infraestrutura de hardware (servidores, nós) e onde o software será instalado.
7. **Profile Diagram**: Usado para estender a própria linguagem UML para domínios específicos.

### B. Diagramas Comportamentais (Behavior Diagrams)
Focam no aspecto dinâmico: o que o sistema faz, fluxos e interações ao longo do tempo.
1. **Use Case Diagram (Casos de Uso)**: Focado em Requisitos Funcionais. Mostra os atores (usuários) e o que eles podem fazer no sistema.
2. **Activity Diagram (Diagrama de Atividades)**: Como um fluxograma avançado, detalha os passos/fluxos de um caso de uso ou lógica de negócio.
3. **State Machine Diagram (Máquina de Estados)**: Modela as mudanças de estado de um único objeto (conectado diretamente com o *State Pattern* do GoF).
4. **Sequence Diagram (Diagrama de Sequência)**: Mostra como os objetos interagem em sequência cronológica (linhas do tempo) durante um processo específico. Muito usado por desenvolvedores e QA.
5. **Communication Diagram**: Destaca o relacionamento e o fluxo de mensagens entre objetos, similar ao de sequência mas focado nos vínculos.
6. **Interaction Overview Diagram**: Uma mistura de diagrama de atividades com diagramas de interação.
7. **Timing Diagram**: Focado fortemente em restrições de tempo real em interações (mais usado em sistemas embarcados).

---

## 4. Ferramentas para Criação
O instrutor focará no uso do **Diagrams.net (Draw.io)** devido aos seus ótimos recursos gratuitos. Outras opções da indústria incluem:
* *Lucidchart* (baseado na web, ótimo para colaboração).
* *Visual Paradigm* (ferramenta robusta focada quase que exclusivamente em modelagem UML).
* *StarUML* (ferramenta open source com extensibilidade e geração de código).
* *Miro* (mais livre, como um quadro branco virtual).

---

## 5. Estendendo a UML (Profiles)
Você não está travado à semântica padrão da UML. Pode customizá-la usando **Profiles** para indústrias específicas (como Saúde ou Finanças, ou os já existentes como **SysML** para engenharia de sistemas e **MARTE** para sistemas embarcados em tempo real).

Os componentes de um *Profile* incluem:
* **Stereotypes (Estereótipos)**: Usado para criar novos elementos (ex: anotar uma classe normal como `<<Service>>` ou `<<Repository>>`).
* **Tagged Values**: Pares chave-valor atrelados aos estereótipos (propriedades adicionais).
* **Constraints**: Regras lógicas estritas (geralmente escritas em *OCL - Object Constraint Language*) determinando o que um elemento pode ou não fazer.
