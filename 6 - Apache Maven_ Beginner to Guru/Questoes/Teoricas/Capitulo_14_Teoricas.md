# Questões Teóricas - Capítulo 14 (Deploying Maven Projects to Nexus)

**1. O Papel do Nexus Repository Manager:** O que é o Sonatype Nexus Repository OSS e por que ele é amplamente considerado uma ferramenta obrigatória na infraestrutura de TI de grandes empresas que desenvolvem em Java?
> [!faq]- 👀 Ver Resposta
> O Nexus é um gerenciador corporativo de repositórios desenvolvido pela Sonatype (os mesmos mantenedores do Maven Central). Ele é essencial porque funciona como um ponto centralizado de controle, armazenamento e governança: hospeda os binários internos gerados pelas equipes, atua como cache intermediário para acelerar o download de dependências externas, bloqueia artefatos com vulnerabilidades de segurança e garante conformidade regulatória sobre tudo o que entra e sai da empresa.

**2. Os Três Tipos Fundamentais de Repositórios no Nexus:** Explique a diferença de finalidade entre os tipos de repositório **Hosted**, **Proxy** e **Group** no Nexus.
> [!faq]- 👀 Ver Resposta
> * **Hosted**: Armazena artefatos gerados e publicados internamente pela própria organização via `mvn deploy` (ex: `nexus-releases` e `nexus-snapshots`).
> * **Proxy**: Conecta-se a um repositório remoto externo (como o Maven Central), fazendo cache local em disco de todas as bibliotecas baixadas pela internet.
> * **Group (Virtual)**: Combina múltiplos repositórios (Hosted e Proxies) sob uma **única URL unificada**, permitindo que os desenvolvedores configurem apenas um endpoint no `settings.xml` para resolver qualquer biblioteca.

**3. Como Funciona o Cache de um Repositório Proxy:** Descreva o ciclo de vida de uma requisição de biblioteca quando um desenvolvedor solicita pela primeira vez um JAR através de um repositório Proxy no Nexus e o que acontece nas requisições subsequentes.
> [!faq]- 👀 Ver Resposta
> 1. **Primeira Requisição**: O desenvolvedor solicita o JAR ao Nexus. O Nexus verifica seu armazenamento interno e nota que não possui o arquivo; conecta-se ao Maven Central pela internet, faz o download do JAR, armazena-o no seu **Blob Store** local e responde ao desenvolvedor.
> 2. **Requisições Subsequentes**: Quando qualquer outro desenvolvedor ou servidor de CI/CD da empresa solicita o mesmo JAR, o Nexus entrega o arquivo instantaneamente a partir de seu disco local na rede interna, sem gastar banda de internet externa.

**4. A Opção "Disable Redeploy" em Repositórios de Release:** Por que a política de implantação de um repositório Hosted de Releases no Nexus deve ser configurada obrigatoriamente com a opção *Disable Redeploy*?
> [!faq]- 👀 Ver Resposta
> Porque garante a **imutabilidade estrita** das versões de produção. Se o redeploy fosse permitido, um desenvolvedor poderia acidentalmente ou maliciosamente sobrescrever o JAR da versão `1.0.0` com um código novo diferente. Isso destruiria a rastreabilidade de bugs e faria com que clientes e servidores recebessem binários diferentes com o mesmo número de versão. Em repositórios de Release, uma vez publicada uma versão, ela nunca pode ser alterada.

**5. Centralização com `<mirrorOf>*</mirrorOf>`:** Quando a equipe de infraestrutura configura o arquivo `settings.xml` de todos os desenvolvedores com um mirror apontando para o Grupo Virtual do Nexus (`nexus-group`), por que os desenvolvedores podem apagar blocos `<repositories>` de seus arquivos `pom.xml`?
> [!faq]- 👀 Ver Resposta
> Porque o espelhamento universal intercepta absolutamente toda requisição de dependência do Maven e a redireciona para o Nexus. Como o Grupo Virtual do Nexus reúne internamente os repositórios de releases internos, snapshots internos e o proxy do Maven Central (e quaisquer outros repositórios adicionais configurados na interface web), o Nexus resolve qualquer dependência de forma transparente sob um único endereço, tornando declarações dispersas de repositórios nos arquivos `pom.xml` totalmente redundantes.

**6. Version Policy (Release, Snapshot e Mixed):** O que é a *Version Policy* em um repositório Nexus e por que a boa prática corporativa recomenda segregar em dois repositórios distintos em vez de utilizar a política *Mixed*?
> [!faq]- 👀 Ver Resposta
> A *Version Policy* dita se o repositório aceita apenas releases estáveis, apenas versões em desenvolvimento (`-SNAPSHOT`) ou ambos (*Mixed*). A boa prática recomenda separá-los em repositórios distintos porque eles possuem ciclos de vida opostos: repositórios de Release exigem imutabilidade, backup crítico de longo prazo e controle rigoroso de acesso, enquanto repositórios de Snapshot são voláteis, sofrem expurgo periódico para economizar espaço e exigem permissão de sobrescrita contínua.

**7. Políticas de Limpeza (*Cleanup Policies*):** Por que um repositório de Snapshots em uma empresa com integração contínua (CI/CD) ativa precisa de uma *Cleanup Policy* configurada no Nexus?
> [!faq]- 👀 Ver Resposta
> Em esteiras de CI/CD com dezenas de deploys diários de múltiplos microsserviços, novos JARs em SNAPSHOT são gerados continuamente. Sem uma política de limpeza, o armazenamento em disco (*Blob Store*) do servidor rapidamente atinge centenas de gigabytes ou terabytes com binários obsoletos de branches temporárias. A *Cleanup Policy* agenda a exclusão automática de snapshots que não foram baixados ou atualizados nos últimos 30 ou 60 dias.

**8. Segurança de Inicialização no Nexus 3 Moderno:** Por que versões recentes do Nexus Repository 3 não utilizam mais a senha padrão histórica `admin/admin123` ao serem instaladas via Docker? Onde a senha inicial temporária é gravada?
> [!faq]- 👀 Ver Resposta
> A senha padrão fixa foi abolida para evitar que instâncias públicas ou corporativas de Nexus ficassem expostas a ataques cibernéticos imediatos caso o administrador esquecesse de alterá-la. Em versões modernas, o Nexus gera uma senha forte randômica no primeiro boot e a salva em um arquivo local seguro dentro do contêiner (`/nexus-data/admin.password`), exigindo que o administrador acesse o arquivo para realizar o primeiro login e force a troca imediata da senha.

**9. Proteção da Cadeia de Suprimentos (Sonatype Nexus Firewall):** Como extensões corporativas como o Nexus Firewall e o Sonatype IQ Server aumentam a segurança da esteira de desenvolvimento de software (*Software Supply Chain Security*)?
> [!faq]- 👀 Ver Resposta
> O Nexus Firewall atua como um antivírus/firewall para dependências de software: ele inspeciona em tempo real todas as bibliotecas solicitadas ao Maven Central pelo repositório Proxy. Se uma dependência solicitada contiver vulnerabilidades críticas conhecidas (CVEs catalogadas), código malicioso (*typosquatting*) ou licenças incompatíveis com as normas jurídicas da empresa, o Nexus bloqueia o download imediatamente e emite um alerta, impedindo que riscos de segurança cheguem às máquinas dos programadores.

**10. Formato de Layout (Strict vs Permissive):** Qual é a diferença entre a *Layout Policy* `Strict` e `Permissive` na criação de um repositório Maven 2 no Nexus?
> [!faq]- 👀 Ver Resposta
> * `Strict`: Exige conformidade rigorosa com o padrão de diretórios e nomenclatura de artefatos do Apache Maven (`/groupId/artifactId/version/artifactId-version.jar`). Se algum arquivo violar a convenção, o upload é rejeitado.
> * `Permissive`: Permite layouts ligeiramente divergentes ou caminhos customizados, sendo útil quando o repositório Maven é compartilhado com ferramentas alternativas que utilizam convenções ligeiramente diferentes (como certas versões do SBT, Ivy ou plugins legados).
