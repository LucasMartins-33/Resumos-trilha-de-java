# Questões Práticas - Capítulo 13 (Deploying Maven Projects to Packagecloud)

🟢 Nível 1: Cadastrando a Conta Gratuita no Packagecloud
Cenário: Você precisa de um repositório Maven remoto na nuvem para hospedar seus artefatos privados de estudo.
Sua Tarefa:
* Acesse a plataforma Packagecloud e crie uma conta gratuita (*Free Tier*).
* Crie dois repositórios na interface web: um chamado `snapshot` e outro chamado `release`.
* Localize o seu **API Token** nas configurações do usuário.

🟡 Nível 2: Adicionando a Extensão Maven Wagon do Packagecloud
Cenário: O Maven padrão não reconhece o protocolo `packagecloud+https://`. Você precisa plugar a extensão de transporte.
Sua Tarefa:
* Abra o `pom.xml` do projeto.
* Na raiz do bloco `<build>`, adicione a seção `<extensions>`.
* Declare a extensão `io.packagecloud.maven.wagon:maven-packagecloud-wagon:0.0.6`.
* Execute `mvn compile` para validar que o Maven baixou a extensão com sucesso.

🟠 Nível 3: Configurando o `<distributionManagement>`
Cenário: Você precisa instruir o Maven sobre onde publicar releases e snapshots no Packagecloud.
Sua Tarefa:
* No `pom.xml`, adicione a seção `<distributionManagement>`.
* Configure a tag `<repository>` com o `<id>packagecloud.release</id>` apontando para a URL:
  `packagecloud+https://packagecloud.io/seu-usuario/release`.
* Configure a tag `<snapshotRepository>` com o `<id>packagecloud.snapshot</id>` apontando para:
  `packagecloud+https://packagecloud.io/seu-usuario/snapshot`.

🔴 Nível 4: Configurando as Credenciais no `settings.xml`
Cenário: O Packagecloud exige autenticação via API Token para aceitar uploads de artefatos.
Sua Tarefa:
* Abra o arquivo `~/.m2/settings.xml`.
* No bloco `<servers>`, adicione dois blocos `<server>`:
  1. `<id>packagecloud.release</id>` com o seu API Token na tag `<password>`.
  2. `<id>packagecloud.snapshot</id>` com o mesmo API Token na tag `<password>`.
* Certifique-se de que os IDs sejam estritamente idênticos aos definidos no `pom.xml`.

🟣 Nível 5: Publicando a Primeira Versão SNAPSHOT
Cenário: Você está desenvolvendo a versão inicial e quer publicá-la para testes de colegas.
Sua Tarefa:
* No `pom.xml`, garanta que a versão termine com `-SNAPSHOT` (ex: `1.0-SNAPSHOT`).
* No terminal, execute: `mvn clean deploy`.
* Analise os logs do console e verifique o upload dos arquivos sendo direcionado para o repositório `snapshot`.
* Abra a interface web do Packagecloud e confira o arquivo JAR publicado.

🟤 Nível 6: Publicando uma Release Oficial Imutável
Cenário: O desenvolvimento da versão 1.0 foi concluído com sucesso e você precisa publicar a release oficial de produção.
Sua Tarefa:
* Altere o `pom.xml` removendo o `-SNAPSHOT`: `<version>1.0</version>`.
* Execute no terminal: `mvn clean deploy`.
* Observe o Maven direcionando o upload para o repositório `release`.
* Na interface web do Packagecloud, verifique que a versão `1.0` está disponível no repositório de releases.

🔵 Nível 7: Testando a Rejeição de Sobrescrita de Release
Cenário: Você quer comprovar que repositórios de release garantem a imutabilidade do código.
Sua Tarefa:
* Sem alterar o número da versão (`1.0`), faça uma pequena alteração no código Java.
* Tente executar `mvn deploy` novamente.
* Analise a resposta de erro retornada pelo repositório remoto recusando a sobrescrita de um pacote de release existente.

🟢 Nível 8: Abrindo o Novo Ciclo de SNAPSHOT
Cenário: Com a versão 1.0 lançada, a equipe retoma o desenvolvimento de novas funcionalidades.
Sua Tarefa:
* Altere o `pom.xml` para `<version>1.1-SNAPSHOT</version>`.
* Execute `mvn clean deploy`.
* Acesse a interface web do Packagecloud e observe os múltiplos snapshots coexistindo com carimbos de data/hora diferentes.

🟡 Nível 9: Consumindo o Pacote Publicado em Outro Projeto
Cenário: Você vai criar um projeto consumidor completamente novo para testar se a biblioteca publicada no Packagecloud pode ser baixada por terceiros.
Sua Tarefa:
* Crie uma nova pasta de projeto com um `pom.xml`.
* Adicione o repositório do Packagecloud no bloco `<repositories>`.
* Declare a dependência para o artefato que você publicou no Nível 6 (`<version>1.0</version>`).
* Execute `mvn clean compile` e comprove o download bem-sucedido a partir do Packagecloud.

🟠 Nível 10: Injetando Credenciais Seguras via Variáveis de Ambiente no CI
Cenário: Você vai subir o projeto para o GitHub e quer que o GitHub Actions ou CircleCI execute o deploy sem expor seu token publicamente.
Sua Tarefa:
* No `settings.xml` do CI, configure o password como `${env.PACKAGECLOUD_TOKEN}`.
* Cadastre o segredo `PACKAGECLOUD_TOKEN` no painel de Environment Variables do seu provedor de CI.
* Configure o comando no pipeline: `mvn deploy -s .circleci/settings.xml`.
* Execute a esteira e confirme que o deploy foi realizado com sucesso sem vazar credenciais no repositório.
