# Questões Teóricas - Capítulo 02: Adicionando JUnit 5 a um Projeto Maven

Testes de fixação sobre gerenciamento de dependências, estrutura de diretórios Maven e execução de testes via Surefire Plugin.

---

### 1. O que são as coordenadas Maven (GroupId e ArtifactId) e qual é o propósito de cada uma na identificação de um projeto?

<details>
<summary>👀 Ver Resposta</summary>

As coordenadas Maven identificam unicamente uma biblioteca ou projeto no ecossistema Java:
* **GroupId:** Representa a organização, empresa ou domínio reverso responsável pelo projeto (por exemplo, `org.junit.jupiter` ou `com.empresa.projeto`). Serve para agrupar projetos relacionados.
* **ArtifactId:** É o identificador específico do módulo ou artefato gerado pelo projeto (por exemplo, `junit-jupiter` ou `servico-pagamento`), diferenciando-o dos demais dentro da mesma organização.
</details>

---

### 2. Por que a convenção de pastas (`src/main/java` e `src/test/java`) deve ser rigorosamente respeitada em um projeto Maven?

<details>
<summary>👀 Ver Resposta</summary>

O Maven baseia-se no princípio de *Convention over Configuration* (Convenção sobre Configuração). Ele pré-configura que o código de produção reside em `src/main/java` e os testes unitários residem em `src/test/java`. Seguir essa convenção permite que o Maven compile, empacote e execute testes automaticamente sem necessidade de configurações adicionais complexas, garantindo ainda que arquivos de teste não sejam incluídos no pacote final (.jar ou .war).
</details>

---

### 3. Qual a vantagem prática de utilizar o artefato agregador `junit-jupiter` em vez de declarar separadamente `junit-jupiter-api`, `junit-jupiter-engine` e `junit-jupiter-params` no `pom.xml`?

<details>
<summary>👀 Ver Resposta</summary>

O artefato agregador `junit-jupiter` atua como uma dependência "guarda-chuva" (umbrella dependency). Ele importa automaticamente em suas dependências transitivas a API (`junit-jupiter-api`), a Engine de execução (`junit-jupiter-engine`) e o módulo de testes parametrizados (`junit-jupiter-params`). Isso simplifica a manutenção do `pom.xml`, evita discrepâncias de versão entre os módulos do JUnit e reduz linhas de configuração redundantes.
</details>

---

### 4. O que significa a diretiva `<scope>test</scope>` na declaração de uma dependência no `pom.xml` e qual o impacto de omiti-la?

<details>
<summary>👀 Ver Resposta</summary>

A tag `<scope>test</scope>` indica que a biblioteca só estará disponível no classpath durante a compilação e execução dos testes (localizados em `src/test/java`). Caso seja omitida, a dependência assumirá o escopo padrão (`compile`), o que fará com que as classes do JUnit e bibliotecas de teste sejam empacotadas no arquivo executável final (.jar) enviado para produção, aumentando desnecessariamente o tamanho do artefato e poluindo o ambiente de produção.
</details>

---

### 5. Qual a diferença operacional entre os comandos `mvn test` e `mvn package` no terminal?

<details>
<summary>👀 Ver Resposta</summary>

* **`mvn test`:** Compila o código-fonte principal e o código de testes e, em seguida, executa exclusivamente a fase de testes unitários do ciclo de vida padrão do Maven.
* **`mvn package`:** É uma fase posterior no ciclo de vida padrão do Maven. Ele executa a compilação, roda os testes unitários e, caso todos os testes passem com sucesso, compila e empacota a aplicação no formato de distribuição final (como um arquivo `.jar` ou `.war` na pasta `target/`).
</details>

---

### 6. Qual é a responsabilidade do `maven-surefire-plugin` no ciclo de vida de build do Maven?

<details>
<summary>👀 Ver Resposta</summary>

O `maven-surefire-plugin` é o plugin padrão do Maven responsável por descobrir, configurar e executar os testes unitários durante a fase `test` do ciclo de vida de build. Ele se comunica com a JUnit Platform para rodar os testes e gera os relatórios em formato texto e XML com os resultados (sucessos, falhas e erros) no diretório `target/surefire-reports`.
</details>

---

### 7. Em quais cenários é necessário configurar explicitamente o `maven-surefire-plugin` no `pom.xml` ao trabalhar com o JUnit 5?

<details>
<summary>👀 Ver Resposta</summary>

Em projetos Java antigos ou sem parent pom moderno, onde a versão padrão herdada do `maven-surefire-plugin` seja inferior à `2.22.0` (ou versões antigas da linha 2.x), o plugin não possui compatibilidade nativa com a JUnit Platform e não consegue detectar os testes do JUnit 5. Nesses casos, é obrigatório declarar explicitamente o plugin na tag `<build>` com uma versão recente (como `3.x`) para que os testes sejam reconhecidos via terminal. Em projetos Spring Boot recentes, essa configuração já é fornecida nativamente pelo starter parent.
</details>

---

### 8. Qual é a utilidade da flag `-Dmaven.test.skip=true` e quais riscos ela traz para o processo de entrega de software?

<details>
<summary>👀 Ver Resposta</summary>

A flag `-Dmaven.test.skip=true` instrui o Maven a ignorar tanto a compilação quanto a execução dos testes durante fases de build (como `mvn package`). Embora útil para acelerar builds puramente experimentais ou de emergência local, seu uso em ambientes de integração contínua (CI/CD) é de alto risco, pois permite que artefatos com lógicas quebradas ou regressões sejam empacotados e publicados em produção sem validação de qualidade.
</details>

---

### 9. O que acontece durante a execução de `mvn test` se uma classe contendo testes JUnit 5 for colocada por engano na pasta `src/main/java` em vez de `src/test/java`?

<details>
<summary>👀 Ver Resposta</summary>

Por padrão, o `maven-surefire-plugin` busca classes de teste apenas dentro do diretório convencionado `src/test/java`. Portanto, a classe de teste em `src/main/java` será tratada como código de produção comum: o plugin ignorará seus métodos de teste durante a fase `test` (eles não serão executados) e o código de teste acabará compilado e embutido no `.jar` final de produção.
</details>

---

### 10. Por que é recomendável acionar o comando de recarregar o projeto (Reload / Sync Maven) na IDE após qualquer alteração no arquivo `pom.xml`?

<details>
<summary>👀 Ver Resposta</summary>

Porque o arquivo `pom.xml` é um arquivo de configuração estático. A IDE precisa ler o novo conteúdo do XML para resolver o grafo de dependências, baixar os novos arquivos `.jar` dos repositórios remotos para o repositório local (`.m2`), atualizar o classpath do módulo e reconfigurar o indexador. Sem o reload, o código continuará apontando erros de compilação como pacotes e anotações inexistentes.
</details>
