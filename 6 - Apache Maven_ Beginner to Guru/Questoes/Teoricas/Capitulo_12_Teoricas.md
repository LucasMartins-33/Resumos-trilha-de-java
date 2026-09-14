# Questões Teóricas - Capítulo 12 (Maven Repositories)

**1. Ordem de Resolução de Dependências:** Descreva passo a passo o algoritmo de busca que o Maven executa quando o código-fonte solicita uma dependência em tempo de compilação.
<details>
<summary>👀 Ver Resposta</summary>

1. O Maven verifica primeiramente o **Repositório Local** em disco (`~/.m2/repository/`). Se o JAR já estiver em cache, utiliza-o imediatamente sem tráfego de rede.
2. Se o artefato não for encontrado localmente, o Maven consulta os **Repositórios Remotos** declarados no `pom.xml` e no `settings.xml` (por padrão o **Maven Central** e quaisquer repositórios adicionais, em ordem de declaração/id).
3. Encontrando o artefato remotamente, realiza o download para o repositório local e prossegue com o build. Se nenhum repositório possuir o artefato, o build falha com `ArtifactResolutionException`.
</details>

**2. O Arquivo `settings.xml` (User vs Global):** Qual é a diferença de escopo entre o arquivo de configurações do usuário (`~/.m2/settings.xml`) e o arquivo de configurações globais (`${maven.home}/conf/settings.xml`)? Qual deles tem precedência?
<details>
<summary>👀 Ver Resposta</summary>

* O **User Settings** (`~/.m2/settings.xml`) aplica-se estritamente ao usuário logado na máquina, sendo o local ideal para credenciais de acesso, tokens privados e personalizações de desenvolvedor.
* O **Global Settings** (`${maven.home}/conf/settings.xml`) reside na instalação do Maven e aplica-se a todos os usuários daquela máquina ou servidor compartilhado.
Em caso de sobreposição de configurações, os valores definidos no **User Settings possuem precedência** e sobrescrevem as diretivas do arquivo global.
</details>

**3. O Conceito de Espelhos (*Repository Mirrors*):** Qual é o objetivo da tag `<mirror>` dentro do `settings.xml` e em quais dois grandes cenários ela é amplamente utilizada na indústria?
<details>
<summary>👀 Ver Resposta</summary>

Um mirror atua como um redirecionador transparente: ele intercepta requisições destinadas a um repositório original e as encaminha para outra URL. Os dois grandes cenários de uso são:
1. **Otimização Geográfica**: Redirecionar o Maven Central para um espelho fisicamente mais próximo do desenvolvedor para acelerar downloads.
2. **Governança Corporativa**: Forçar que todas as requisições do Maven (`<mirrorOf>*</mirrorOf>`) passem obrigatoriamente pelo servidor de repositório interno da empresa (Nexus ou Artifactory), impedindo que máquinas acessem a internet pública diretamente.
</details>

**4. A Tag `<mirrorOf>` e a Expressão `external:*`:** Qual é a diferença crucial entre configurar `<mirrorOf>*</mirrorOf>` e configurar `<mirrorOf>external:*</mirrorOf>` no `settings.xml`?
<details>
<summary>👀 Ver Resposta</summary>

* `<mirrorOf>*</mirrorOf>` intercepta absolutamente **todas** as requisições de repositório, inclusive servidores de teste locais em memória (`localhost` ou protocolo `file://`).
* `<mirrorOf>external:*</mirrorOf>` intercepta qualquer repositório remoto na rede externa, mas **exclui** repositórios locais ou conexões de loopback (`localhost`). Essa é a prática moderna recomendada para evitar que suítes de testes automatizados que sobem servidores locais sejam quebradas pelo espelhamento.
</details>

**5. Políticas de Atualização e Verificação (`<updatePolicy>` e `<checksumPolicy>`):** O que definem as tags `<updatePolicy>` e `<checksumPolicy>` dentro da configuração de um `<repository>` no POM?
<details>
<summary>👀 Ver Resposta</summary>

* `<updatePolicy>`: Define a frequência com que o Maven checa se há versões mais novas do artefato no repositório remoto. Valores possíveis: `always` (a cada build), `daily` (padrão, uma vez ao dia), `interval:X` (a cada X minutos) ou `never`.
* `<checksumPolicy>`: Define como o Maven reage se o arquivo baixado não bater com o hash SHA-1/MD5 publicado no repositório. Valores: `fail` (aborta o build imediatamente por segurança), `warn` (emite um aviso no console e continua) ou `ignore`.
</details>

**6. Instalação Manual de Dependências (`install:install-file`):** Em quais situações um desenvolvedor precisa recorrer ao comando `mvn install:install-file` e qual é o efeito colateral dessa prática em ambientes de equipe?
<details>
<summary>👀 Ver Resposta</summary>

É utilizado quando uma biblioteca externa, driver legado ou arquivo proprietário não existe em nenhum repositório Maven público ou corporativo e não pode ser distribuído publicamente por restrições contratuais. O efeito colateral é que o artefato é registrado **estritamente no cache local daquela máquina**. Em servidores de CI/CD ou na máquina de colegas de equipe, o build falhará imediatamente a menos que o comando seja repetido em cada computador ou que o artefato seja publicado em um gerenciador corporativo compartilhado.
</details>

**7. Arquitetura de Criptografia de Senhas no Maven:** Por que é desaconselhado salvar senhas de servidores em texto puro no `settings.xml` e qual é a função do arquivo `settings-security.xml`?
<details>
<summary>👀 Ver Resposta</summary>

Salvar senhas em texto puro viola padrões de segurança corporativa e expõe credenciais caso o arquivo seja compartilhado ou acessado indevidamente. O Maven resolve isso com uma chave criptográfica mestra: o comando `mvn --encrypt-master-password` gera uma chave armazenada no arquivo `~/.m2/settings-security.xml`. O comando `mvn --encrypt-password` utiliza essa chave mestra para criptografar as senhas de repositório, permitindo que apenas hashes cifrados (ex: `{jSMO...}`) sejam inseridos no `settings.xml`.
</details>

**8. Vinculação entre POM e Settings através do `<id>`:** Como o Maven sabe exatamente qual usuário e senha utilizar no momento de fazer o download ou upload de um artefato em um repositório remoto autenticado?
<details>
<summary>👀 Ver Resposta</summary>

O Maven realiza essa associação através da correspondência exata de identificadores: o valor da tag `<id>` definido no `<repository>` (no `pom.xml`) ou no `<distributionManagement>` **deve ser idêntico** ao valor da tag `<id>` declarado dentro do bloco `<server>` no arquivo `settings.xml`. Se os IDs não forem estritamente iguais, o Maven não injetará as credenciais e a requisição resultará em erro de autenticação (`401 Unauthorized`).
</details>

**9. O Fim do JCenter e a Consolidação do Maven Central:** O que levou ao fechamento do repositório JCenter pela JFrog e qual impacto isso causou na comunidade Java e Android?
<details>
<summary>👀 Ver Resposta</summary>

O JCenter foi descontinuado pela JFrog em 2021 devido a custos operacionais e consolidação estratégica de mercado. Como ele era o repositório padrão do Gradle e de projetos Android, a indústria sofreu com builds quebrados em projetos não atualizados. A comunidade consolidou o **Maven Central** como o repositório padrão universal e o Google criou o **Google Maven Repository** (`maven.google.com`) para hospedar artefatos do Android.
</details>

**10. A Mudança de Licenciamento do Oracle JDBC:** Historicamente, por que desenvolvedores sofriam para adicionar o driver `ojdbc` ao Maven e qual é a situação atual dessa biblioteca no ecossistema moderno?
<details>
<summary>👀 Ver Resposta</summary>

Historicamente, a Oracle exigia a aceitação manual de termos contratuais (*Click-Through License*) no portal OTN, proibindo a inclusão pública do driver JDBC no Maven Central. Isso forçava desenvolvedores a baixar o JAR manualmente e executar `mvn install:install-file` ou configurar o repositório autenticado `maven.oracle.com`. A partir de 2019 (versão 19.3+), a Oracle adotou a licença permissiva FUTC e disponibilizou oficialmente e gratuitamente todos os drivers modernos (`ojdbc8`, `ojdbc11`, `ojdbc17`) diretamente no **Maven Central**, eliminando a necessidade de credenciais ou instalações manuais.
</details>
