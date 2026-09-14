# Questões Práticas - Capítulo 14 (Deploying Maven Projects to Nexus)

🟢 Nível 1: Inicializando o Sonatype Nexus 3 via Docker
Cenário: Você precisa de um servidor Nexus local para simular a infraestrutura de repositórios de uma grande corporação.
Sua Tarefa:
* Abra o terminal e execute o Nexus 3 em background mapeando a porta 8081:
  `docker run -d -p 8081:8081 --name nexus-local sonatype/nexus3`.
* Aguarde de 1 a 2 minutos para a inicialização completa da aplicação Java.
* Abra o navegador em `http://localhost:8081` e verifique a tela de boas-vindas do Nexus Repository Manager.

🟡 Nível 2: Recuperando a Senha e Autenticando no Nexus
Cenário: Versões modernas do Nexus não aceitam a senha padrão histórica `admin123` e exigem a leitura da senha temporária gerada no contêiner.
Sua Tarefa:
* No terminal, recupere a senha gerada dentro do contêiner Docker:
  `docker exec -it nexus-local cat /nexus-data/admin.password`.
* Clique em *Sign In* no topo direito da tela do Nexus, informe o usuário `admin` e cole a senha.
* Siga o assistente de configuração definindo uma nova senha definitiva e ativando o acesso anônimo (*Enable Anonymous Access*).

🟠 Nível 3: Criando o Repositório Hosted de Snapshots
Cenário: Você precisa de um repositório no Nexus para armazenar os pacotes de desenvolvimento dos microsserviços da sua empresa.
Sua Tarefa:
* Na interface do Nexus, clique no ícone da engrenagem (*Server administration*) e vá em *Repositories* ➔ *Create repository*.
* Selecione a receita **`maven2 (hosted)`**.
* Nomeie o repositório como `nexus-snapshot`.
* Na política de versão (*Version policy*), selecione **Snapshot**.
* Deixe a política de layout em *Strict* e salve a criação.

🔴 Nível 4: Criando o Repositório Hosted de Releases com Imutabilidade
Cenário: Você vai criar o repositório oficial de produção onde novas versões só podem ser enviadas uma única vez.
Sua Tarefa:
* Crie outro repositório com a receita **`maven2 (hosted)`** chamado `nexus-release`.
* Na política de versão, selecione **Release**.
* Na seção *Deployment policy*, escolha obrigatoriamente **Disable redeploy**.
* Salve o repositório.

🟣 Nível 5: Configurando o Projeto para Publicar no Nexus
Cenário: Você precisa apontar o `distributionManagement` do seu projeto Maven para as URLs locais do Nexus.
Sua Tarefa:
* Abra o `pom.xml` do projeto e configure `<distributionManagement>`:
  * `<repository>`: `<id>nexus-release</id>` com URL `http://localhost:8081/repository/nexus-release/`.
  * `<snapshotRepository>`: `<id>nexus-snapshot</id>` com URL `http://localhost:8081/repository/nexus-snapshot/`.
* No `~/.m2/settings.xml`, adicione as credenciais de `admin` para ambos os servidores (`nexus-release` e `nexus-snapshot`).

🟤 Nível 6: Executando Deploys de Snapshot e Release no Nexus
Cenário: Você vai testar a publicação prática de ambas as versões no seu servidor Nexus local.
Sua Tarefa:
* Defina a versão no `pom.xml` como `1.0-SNAPSHOT` e execute: `mvn clean deploy`.
* Na interface do Nexus, navegue até *Browse* ➔ `nexus-snapshot` e confirme a presença do arquivo.
* Altere a versão para `1.0` e execute `mvn clean deploy`.
* Navegue até *Browse* ➔ `nexus-release` e confirme o JAR oficial publicado.

🔵 Nível 7: Criando um Grupo Virtual de Repositórios (`maven2 group`)
Cenário: Os desenvolvedores querem uma URL única para baixar dependências externas do Maven Central e artefatos internos ao mesmo tempo.
Sua Tarefa:
* No Nexus, crie um novo repositório com a receita **`maven2 (group)`** chamado `nexus-group`.
* No painel de membros do grupo (*Group members*), adicione os repositórios na seguinte ordem de prioridade:
  1. `nexus-release`
  2. `nexus-snapshot`
  3. `maven-central` (proxy já embutido no Nexus)
* Salve a criação e copie a URL pública gerada para o grupo.

🟢 Nível 8: Espelhamento Universal no `settings.xml`
Cenário: Você vai configurar sua máquina para rotear 100% dos downloads através do Grupo Virtual do Nexus.
Sua Tarefa:
* Abra o `~/.m2/settings.xml` e crie a seção `<mirrors>`.
* Configure:
  `<mirror><id>nexus-mirror</id><url>http://localhost:8081/repository/nexus-group/</url><mirrorOf>*</mirrorOf></mirror>`.
* Salve o arquivo.

🟡 Nível 9: Comprovando o Cache Local do Repositório Proxy
Cenário: Você quer visualizar na prática o Nexus fazendo o download da internet apenas uma vez e servindo as requisições seguintes direto do cache.
Sua Tarefa:
* No projeto, adicione uma dependência que você nunca baixou antes (ex: uma versão antiga do H2).
* Desative temporariamente o cache da sua IDE e execute `mvn package` no terminal.
* Observe o console indicando download a partir de `http://localhost:8081/...`.
* No Nexus, vá em *Browse* ➔ `maven-central` e verifique que o JAR agora está armazenado no disco do Nexus.
* Apague o JAR do seu `~/.m2/repository` e rode `mvn package` novamente: veja que o download agora é quase instantâneo porque veio da rede local do Nexus.

🟠 Nível 10: Configurando uma Política de Limpeza (*Cleanup Policy*)
Cenário: O repositório de snapshots está crescendo descontroladamente e você precisa agendar a limpeza automática de artefatos velhos.
Sua Tarefa:
* Na administração do Nexus, vá em *Repository* ➔ *Cleanup Policies* e clique em *Create Cleanup Policy*.
* Selecione o formato `maven2` e configure a remoção de componentes que não foram baixados nos últimos 30 dias (*Last downloaded > 30 days*).
* Vincule essa política ao repositório `nexus-snapshot` na tela de edição do repositório.
