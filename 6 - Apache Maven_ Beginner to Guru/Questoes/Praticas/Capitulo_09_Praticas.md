# Questões Práticas - Capítulo 09 (Testing with Maven)

🟢 Nível 1: Escrevendo o Primeiro Teste com JUnit 5 Jupiter
Cenário: Você precisa criar uma suíte moderna de testes unitários para a classe `CalculadoraService`.
Sua Tarefa:
* Adicione a dependência `org.junit.jupiter:junit-jupiter-api:5.10.0` e `junit-jupiter-engine:5.10.0` com escopo `test`.
* Em `src/test/java/com/minhaempresa/CalculadoraServiceTest.java`, crie um método anotado com `@Test` do Jupiter.
* Use `Assertions.assertEquals(4, service.somar(2, 2))` para validar a lógica.
* Execute `mvn test` e analise o sumário de testes do Surefire no console.

🟡 Nível 2: Executando Testes Legados com JUnit Vintage
Cenário: Você herdou uma classe de teste antiga escrita em JUnit 4 com a anotação `org.junit.Test`, e o Maven moderno precisa executá-la sem que você tenha tempo de refatorar.
Sua Tarefa:
* Crie um arquivo `src/test/java/com/minhaempresa/LegadoTest.java` utilizando a anotação antiga `@org.junit.Test` do JUnit 4.
* Adicione no `pom.xml` a dependência do motor de compatibilidade: `org.junit.vintage:junit-vintage-engine:5.10.0` com escopo `test`.
* Execute `mvn test`.
* Comprove no log que o Surefire executou tanto o teste novo em JUnit 5 quanto o teste legado em JUnit 4 na mesma bateria.

🟠 Nível 3: Filtrando Execuções com Tags do JUnit 5 (`@Tag`)
Cenário: Alguns testes unitários são lentos e você deseja executá-los apenas quando solicitado, separando-os dos testes rápidos do dia a dia.
Sua Tarefa:
* Anote um teste com `@Tag("lento")` e outro com `@Tag("rapido")`.
* No terminal, execute apenas os testes rápidos instruindo o Surefire:
  `mvn test -Dgroups=rapido`.
* Execute o comando inverso para rodar apenas os lentos: `mvn test -Dgroups=lento`.

🔴 Nível 4: Configurando Testes de Integração com o Failsafe
Cenário: Você precisa configurar o projeto para rodar testes de integração que simulam chamadas de rede ou banco de dados.
Sua Tarefa:
* Crie a classe `src/test/java/com/minhaempresa/ServicoExternoIT.java` seguindo o padrão de nomenclatura `*IT.java`.
* Configure o `maven-failsafe-plugin` no `pom.xml` vinculando os goals `integration-test` e `verify`.
* Execute `mvn test` e comprove que o teste `*IT` foi pulado.
* Execute `mvn verify` e comprove que ele foi executado.

🟣 Nível 5: Pular Apenas Testes de Integração (`-DskipITs`)
Cenário: Durante o desenvolvimento local rápido, você quer rodar todos os testes unitários do Surefire, mas deseja pular os testes lentos de integração do Failsafe.
Sua Tarefa:
* No terminal, execute: `mvn verify -DskipITs`.
* Observe a saída do console: o Surefire executa `CalculadoraServiceTest`, mas o Failsafe ignora `ServicoExternoIT`, completando o build rapidamente.

🟤 Nível 6: Gerando Relatórios HTML com `surefire-report-plugin`
Cenário: O líder técnico solicitou um relatório visual em HTML com o histórico de execução de testes para enviar à equipe de QA.
Sua Tarefa:
* No terminal, execute o goal avulso de relatório: `mvn surefire-report:report`.
* Navegue até o diretório `target/site/`.
* Abra o arquivo `surefire-report.html` em um navegador web e inspecione a tabela de tempos, falhas e percentual de sucesso.

🔵 Nível 7: Coleta de Cobertura de Código com JaCoCo
Cenário: Você precisa auditar quantas linhas de código da sua aplicação estão sendo de fato exercitadas pela suíte de testes.
Sua Tarefa:
* Adicione o plugin `org.jacoco:jacoco-maven-plugin:0.8.11` no `pom.xml`.
* Configure a execução do goal `prepare-agent` (antes dos testes).
* Configure a execução do goal `report` na fase `test` ou `verify`.
* Execute `mvn clean verify`.
* Abra o arquivo `target/site/jacoco/index.html` e analise a cobertura de linhas e ramos condicionais.

🟢 Nível 8: Impondo Cobertura Mínima Obrigatória (`jacoco:check`)
Cenário: Para evitar que novos desenvolvedores subam código sem testes, o build deve quebrar se a cobertura geral de código for inferior a 80%.
Sua Tarefa:
* No `jacoco-maven-plugin`, adicione uma execução com o goal `check`.
* Configure uma regra de limite exigindo:
  `<limit><counter>LINE</counter><value>COVEREDRATIO</value><minimum>0.80</minimum></limit>`.
* Execute `mvn verify`.
* Crie intencionalmente uma nova classe com métodos sem testes e rode `mvn verify` novamente para observar o build falhar acusando violação de cobertura mínima.

🟡 Nível 9: Análise Estática com SpotBugs
Cenário: Você quer detectar potenciais vazamentos de recursos ou comparações erradas de objetos antes de colocar o código em produção.
Sua Tarefa:
* Adicione o plugin `com.github.spotbugs:spotbugs-maven-plugin:4.8.2.0`.
* Escreva um método na classe `Main` contendo um erro clássico (ex: comparar duas Strings com `==` ou abrir um `FileInputStream` e esquecer de fechar).
* Execute no terminal: `mvn spotbugs:check`.
* Observe o Maven interromper o build com `BUILD FAILURE` e apontar o número da linha com o bug em potencial.

🟠 Nível 10: Testes BDD Parametrizados com Spock no JUnit Platform
Cenário: Você precisa testar uma tabela com 5 cenários diferentes de cálculo de frete sem duplicar código de teste.
Sua Tarefa:
* Crie a especificação `src/test/groovy/com/minhaempresa/FreteSpec.groovy`.
* Utilize a anotação ou bloco `where:` do Spock para criar uma tabela de dados:
  `peso | distancia || freteEsperado`
  ` 5   |    10     ||     15.0     `
  `10   |    50     ||     45.0     `
* Execute `mvn test` e observe o Surefire rodando cada linha da tabela como um teste individual.
