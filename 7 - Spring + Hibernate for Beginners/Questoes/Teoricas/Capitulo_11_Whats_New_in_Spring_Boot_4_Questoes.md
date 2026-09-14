# Questões Teóricas - Capítulo 11: What's New in Spring Boot 4

### Questão 1
Em qual versão do Spring Framework o **Spring Boot 4** é baseado e qual é o requisito mínimo de versão da linguagem Java?

> [!faq]- Resposta
> O Spring Boot 4 é baseado no **Spring Framework 7**. A versão mínima do Java exigida é o **Java 17**, embora o uso do **Java 21** ou **Java 25** (versões LTS) seja fortemente recomendado.

---

### Questão 2
Quais foram os 4 principais pilares estabelecidos pela equipe do Spring para o desenvolvimento do Spring Boot 4?

> [!faq]- Resposta
> 1. **Faster**: Inicialização mais rápida e menor consumo de memória RAM.
> 2. **Leaner**: Design modularizado para reduzir o inchaço do classpath.
> 3. **Safer**: Suporte nativo à segurança contra valores nulos (*Null Safety*) com JSpecify e atualizações de runtime.
> 4. **Cloud Readiness**: Probes nativos do Kubernetes configurados por padrão e observabilidade aprimorada.

---

### Questão 3
Por que a migração do Spring Boot 3 para o Spring Boot 4 é considerada infinitamente mais simples do que a migração do Spring Boot 2 para o Spring Boot 3?

> [!faq]- Resposta
> A migração do Spring Boot 2 para o 3 envolveu uma grande *breaking change* de renomeação de pacotes de toda a especificação Java de `javax.*` para `jakarta.*`, impactando até 50% da base de código dos projetos. Já a migração para o Spring Boot 4 é uma **atualização incremental**, mantendo 100% dos conceitos, anotações de Injeção de Dependências e padrões do Spring Boot 3 válidos.

---

### Questão 4
Como funciona o recurso nativo de **First-Class API Versioning** no Spring Boot 4?

> [!faq]- Resposta
> O Spring Boot 4 permite versionar APIs REST nativamente declarando a propriedade `spring.mvc.api-version.use-path-segment=1` no `application.properties` e mapeando as versões nos controladores via atributo `version` nas anotações de mapeamento (ex: `@GetMapping(version = "1")`). O framework extrai e mapeia a versão automaticamente da URL sem código ou filtros customizados.

---

### Questão 5
Qual é o novo pacote oficial das classes da biblioteca **Jackson 3** no Spring Boot 4 e como isso afeta projetos existentes?

> [!faq]- Resposta
> No Jackson 3 (biblioteca JSON padrão do Spring Boot 4), as classes foram movidas do pacote `com.fasterxml.jackson.databind.*` para **`tools.jackson.databind.*`**. Isso afeta apenas classes do controller/serviço que importem ou manipulem classes nativas do Jackson manualmente (como no processamento manual de requisições `PATCH`).

---

### Questão 6
Qual é a nova classe/interface preferencial do Jackson 3 injetada pelo Spring Boot 4 em substituição ao clássico `ObjectMapper`?

> [!faq]- Resposta
> A classe preferencial para manipulação de JSON no Jackson 3 / Spring Boot 4 é o **`JsonMapper`**, que é autoconfigurado como um Bean Spring e pode ser injetado via construtor no controller.

---

### Questão 7
O que é o **JSpecify** e como ele é integrado ao Spring Boot 4?

> [!faq]- Resposta
> JSpecify é um padrão de anotações Java (`@Nullable`, `@NonNull`) para suporte de *Null Safety*. No Spring Boot 4, as APIs internas do framework vêm anotadas com JSpecify, permitindo que IDEs e linters detectem riscos de `NullPointerException` antes de rodar a aplicação. O uso nas classes do desenvolvedor é opcional.

---

### Questão 8
Qual starter Maven da camada Web foi depreciado no Spring Boot 4 e qual é o seu substituto oficial?

> [!faq]- Resposta
> O starter `spring-boot-starter-web` foi depreciado e substituído pelo novo starter **`spring-boot-starter-web-mvc`**.

---

### Questão 9
Qual starter Maven da camada AOP foi alterado no Spring Boot 4?

> [!faq]- Resposta
> O starter `spring-boot-starter-aop` foi renomeado e atualizado para **`spring-boot-starter-aspectj`**.

---

### Questão 10
Como o método `updateValue()` da nova API `JsonMapper` do Jackson 3 simplifica o tratamento de exceções em requisições HTTP `PATCH` em comparação ao Jackson 2?

> [!faq]- Resposta
> No Jackson 3, o método `jsonMapper.updateValue(objetoAlvo, mapaComCamposPatch)` lança exceções não-checadas (*unchecked exceptions*). Isso elimina a necessidade de declarar a cláusula `throws JsonMappingException` na assinatura do método do controlador REST.
