# Questões Teóricas - Capítulo 04: Adicionando JUnit 5 a Projetos Java Básicos

Testes de fixação sobre configuração manual de bibliotecas na IDE, estrutura de pacotes, visibilidade package-private e asserção de falha intencional.

---

### 1. O que significa marcar um diretório como "Test Sources Root" no IntelliJ IDEA e qual o efeito prático dessa ação?

<details>
<summary>👀 Ver Resposta</summary>

Marcar um diretório como "Test Sources Root" instrui a IDE de que aquele diretório contém código-fonte dedicado exclusivamente a testes automatizados. O IntelliJ altera a cor visual da pasta (geralmente para verde), aplica regras de compilação diferenciadas, habilita atalhos de geração de código de teste e assegura que esses arquivos não sejam incluídos no pacote de produção da aplicação.
</details>

---

### 2. Por que é considerada uma boa prática manter a classe de teste no mesmo nome de pacote (`package`) da classe de produção correspondente?

<details>
<summary>👀 Ver Resposta</summary>

Manter o mesmo pacote (por exemplo, `com.empresa.service` para a classe de produção em `src/main/java` e para a classe de teste em `src/test/java`) permite que a classe de teste acesse métodos e membros da classe sob teste que possuem o modificador de acesso padrão (*package-private*), sem forçar o desenvolvedor a abrir a visibilidade desses métodos para `public` apenas para viabilizar os testes. Além disso, melhora a organização e localização visual dos testes na árvore de pacotes.
</details>

---

### 3. Qual é o comportamento padrão do JUnit 5 quando um método anotado com `@Test` é executado sem nenhuma asserção e sem lançar nenhuma exceção?

<details>
<summary>👀 Ver Resposta</summary>

No JUnit 5, se um método anotado com `@Test` completa seu fluxo de execução sem disparar nenhuma exceção não tratada e sem falhar em nenhuma asserção, o teste é considerado aprovado (**verde / passed**), mesmo que o corpo do método esteja completamente vazio. O JUnit assume que a ausência de exceções indica sucesso.
</details>

---

### 4. Qual é a função do método estático `fail()` da classe `Assertions` no JUnit 5 e em que situações ele deve ser utilizado?

<details>
<summary>👀 Ver Resposta</summary>

O método `fail(String message)` força imediatamente a reprovação (**vermelho / failed**) do teste unitário com a mensagem descritiva fornecida, lançando um `AssertionFailedError`. Ele é útil em cenários de desenvolvimento orientado a testes para sinalizar que um teste ainda não foi implementado ("TODO / Stub"), ou em blocos de tratamento manual onde um determinado trecho de código jamais deveria ser alcançado.
</details>

---

### 5. No Eclipse IDE, por que o diretório de testes deve ser criado como uma "Source Folder" em vez de uma pasta comum ("Folder")?

<details>
<summary>👀 Ver Resposta</summary>

No Eclipse, uma pasta comum ("Folder") é tratada apenas como um contêiner de arquivos de dados ou recursos não compilados. Uma "Source Folder" é formalmente reconhecida pelo Java Builder do Eclipse como raiz de compilação de código Java, fazendo com que todo arquivo `.java` contido nela seja compilado e colocado no *Build Path* do projeto.
</details>

---

### 6. Como IDEs modernas como o IntelliJ ou Eclipse resolvem a dependência do JUnit 5 em projetos Java puros que não utilizam Maven ou Gradle?

<details>
<summary>👀 Ver Resposta</summary>

Através de assistentes automatizados (como a opção "Fix" na lâmpada de contexto do IntelliJ ou "Add to Build Path" no Eclipse). A IDE faz o download dos arquivos binários `.jar` do JUnit Platform e JUnit Jupiter de um repositório central ou utiliza bibliotecas empacotadas internamente pela própria IDE, anexando esses JARs diretamente ao *Classpath/Build Path* das propriedades do módulo do projeto.
</details>

---

### 7. Por que os métodos e classes de teste no JUnit 5 não precisam mais ser declarados como `public` (diferente do JUnit 4)?

<details>
<summary>👀 Ver Resposta</summary>

No JUnit 4, a reflexão exigia que as classes e métodos de teste fossem estritamente `public` para que o framework pudesse invocá-los. No JUnit 5, o Jupiter foi redesenhado para utilizar reflexão moderna do Java, permitindo que classes e métodos de teste tenham visibilidade *package-private* (modificador padrão, sem palavra-chave). Isso deixa o código mais limpo e conciso, mantendo `public` apenas o que realmente precisa ser exposto para fora do pacote.
</details>

---

### 8. Quais são as principais desvantagens e riscos de gerenciar dependências de teste manualmente pela IDE em vez de usar uma ferramenta de build como Maven ou Gradle?

<details>
<summary>👀 Ver Resposta</summary>

1. **Falta de Portabilidade:** As dependências ficam atreladas aos arquivos proprietários de configuração daquela IDE específica (`.iml`, `.classpath`), dificultando o compartilhamento com colegas que utilizem outra ferramenta.
2. **Incompatibilidade com CI/CD:** Servidores de integração contínua (como GitHub Actions, Jenkins, GitLab CI) executam em ambientes headless (sem interface gráfica de IDE) e dependem de comandos de build de terminal como `mvn test` ou `./gradlew test` para compilar e rodar os testes.
3. **Gestão de Versões Caótica:** A atualização manual de versões de JARs exige intervenção arquivo por arquivo, sujeita a conflitos de classpath.
</details>

---

### 9. O que significa a icônica convenção da "Barra Verde" e da "Barra Vermelha" nas ferramentas de execução gráfica de testes de IDEs?

<details>
<summary>👀 Ver Resposta</summary>

É o padrão visual universal de feedback de testes automatizados:
* **Barra Verde:** Todos os testes unitários da suíte foram executados e todas as asserções e regras foram satisfeitas com sucesso (zero falhas e zero erros).
* **Barra Vermelha:** Pelo menos um teste da suíte falhou em uma asserção (`AssertionFailedError`) ou lançou uma exceção inesperada, exigindo atenção e correção imediata por parte do desenvolvedor.
</details>

---

### 10. Em um projeto Java puro, o que acontece se o pacote declarado na classe de teste divergir do caminho físico de diretórios onde o arquivo `.java` está gravado?

<details>
<summary>👀 Ver Resposta</summary>

O compilador Java (`javac`) gerará um erro de compilação obrigatório (*Package name does not correspond to the file path*). No Java, a instrução `package` no topo do arquivo `.java` deve espelhar estritamente a hierarquia real de pastas físicas dentro do *Source Folder*, independentemente de ser código de produção ou de teste.
</details>
