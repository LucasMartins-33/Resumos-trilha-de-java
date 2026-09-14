# Questões Teóricas - Capítulo 24 (Java Database Connectivity - JDBC)

**1. O Paradigma do JDBC:** O que significa a sigla JDBC, o que ela representa tecnologicamente, e qual a analogia do JDBC como o "inglês" no mundo dos bancos de dados?
<details>
<summary>👀 Ver Resposta</summary>

JDBC (Java Database Connectivity) é uma interface (API) de bibliotecas do Java. A analogia é que cada Banco de Dados do mercado (Oracle, SQL Server, MySQL, Postgres) fala uma língua "física" completamente diferente e caótica nos bastidores. O JDBC atua como o "inglês": uma língua padrão unificada em que você escreve o código no Java, e ela se encarrega de se virar para enviar a mensagem correta pro banco.
</details>

**2. A Chave Universal (Vendor Driver):** Como o JDBC é um padrão cego do Java, ele precisa de uma ajuda especializada para conseguir falar com os bancos. Qual arquivo e qual a função do "Driver de Banco de Dados" (ex: MySQL Connector .jar)?
<details>
<summary>👀 Ver Resposta</summary>

O Driver é o arquivo ".jar" fornecido fisicamente pela criadora do Banco (ex: a Microsoft ou a Oracle). Ele é o tradutor oficial (dicionário). A gente acopla esse `.jar` na pasta `/lib` e no `Build Path` do nosso projeto. Sem ele o JDBC não consegue traduzir a conexão.
</details>

**3. O Mapa (JDBC URL String):** O que é uma JDBC URL (como a String `jdbc:mysql://127.0.0.1:3306/meubanco`) e o que compõem suas 3 partes principais após o protocolo de rede?
<details>
<summary>👀 Ver Resposta</summary>

É a localização (o mapa de endereços) do banco. Após a declaração da arquitetura (`jdbc:mysql://`), ela possui o IP/Hospedeiro (ex: `localhost` ou `127.0.0.1`), a porta oficial de conexão (como `3306` do mysql ou `5432` do postgres) e, no fim, o nome da base de dados (`/nome_banco`).
</details>

**4. A Tríade da Conexão:** Explique sucintamente o papel dos 3 objetos vitais e obrigatórios importados do pacote `java.sql` para conseguirmos injetar queries (`Connection`, `Statement`, `ResultSet`).
<details>
<summary>👀 Ver Resposta</summary>

1) `Connection`: A ponte de conexão validada com usuário/senha. 2) `Statement`: O carteiro/objeto veicular que nós recheamos com o comando SQL ("SELECT...") e engatilhamos via Java. 3) `ResultSet`: O balde receptor; ele capta a tabela (linhas e colunas) que o banco de dados devolve como resposta e armazena pra gente processar no Java.
</details>

**5. Lendo Dados vs Alterando Dados:** Qual a diferença imperativa e fundamental entre a invocação do método `.executeQuery()` e do método `.executeUpdate()` a partir de um Statement?
<details>
<summary>👀 Ver Resposta</summary>

O `executeQuery` é de "Leitura Pura" e se usa em conjunto APENAS com o comando SQL `SELECT`. Ele retorna um `ResultSet` (tabela de dados pesada). O `executeUpdate` é usado com ordens de mutação/agressão (`INSERT`, `UPDATE` ou `DELETE`). Por natureza, ele NÃO retorna dados, retorna apenas um número `int` informando quantas linhas da tabela de lá foram afetadas pelo ataque.
</details>

**6. Percorrendo o Tabuleiro (Result Set Loop):** Como o objeto `ResultSet` armazena múltiplas linhas vindas do banco, nós utilizamos um comando `while` em conjunto com qual método para fatiar e imprimir linha a linha dos resultados?
<details>
<summary>👀 Ver Resposta</summary>

Usamos o laço condicional: `while (resultSet.next()) { ... }`. O método `.next()` é um ponteiro mágico. Ele pula para a linha de baixo do arquivo. Se ele encontrar dados naquela linha, ele devolve `true` e roda o loop. Quando as linhas da tabela acabam, ele devolve `false` e encerra o loop de leitura.
</details>

**7. A Máquina de Tipos (`getInt`, `getString`):** Dentro do laço de repetição do ResultSet, de que maneira específica o Java "extrai" os valores das colunas daquela determinada linha para o nosso código base?
<details>
<summary>👀 Ver Resposta</summary>

O Java nos fornece métodos altamente estritos de tipagem como `resultSet.getString("nome_coluna")` para extrair letras ou `resultSet.getInt("idade_coluna")` para extrair números matemáticos, garantindo que o dado do SQL seja moldado com total segurança para as variáveis primitivas e classes do Java nativo.
</details>

**8. O Perigo da Atualização sem Mira (Where):** Em SQL puro invocado via JDBC (usando `executeUpdate`), qual é o perigo brutal de rodar comandos de `UPDATE` ou `DELETE` e qual instrução de contenção de dano sempre usamos no final da query String?
<details>
<summary>👀 Ver Resposta</summary>

Sem a proteção devida, comandos brutos como `UPDATE funcionarios SET salario = 10` ou `DELETE FROM funcionarios` alterarão ou incinerarão as contas e vidas de ABSOLUTAMENTE TODOS OS FUNCIONÁRIOS da tabela ao mesmo tempo. Sempre deve ser usada a barreira do `WHERE` (Ex: `WHERE id = 5`) limitando cirurgicamente as ações a linhas identificáveis.
</details>

**9. O Castigo das Falhas (SQLException):** Se a senha do banco de dados estiver errada no JDBC, ou a Query contiver erros de digitação (ex: `SELECT * FROMMM tabelax`), qual erro e qual ação o Java tomará? Como blindamos isso?
<details>
<summary>👀 Ver Resposta</summary>

Qualquer problema na ponte entre o Java e o SQL joga agressivamente a perigosa *Checked Exception* chamada `SQLException`. Como o Java prevê que conversar com o mundo externo é trágico, todo e qualquer script de JDBC obriga a criação de um forte bloco Try/Catch capturando o `SQLException` na raiz.
</details>

**10. A Limpeza de Outono:** No fim do processamento do Banco de Dados em código de empresa antiquado sem *Try-with-Resources*, qual rotina devemos religiosamente acoplar dentro dos blocos de limpeza (`finally`) com as variáveis de Connection e Statement e por quê?
<details>
<summary>👀 Ver Resposta</summary>

Devemos usar `.close()` em cascata na *Connection*, no *Statement* e no *ResultSet*. Conexões de banco de dados são como pontes pesadas no SO. Se não fechadas pelo dev, elas consumirão *Pools de Conexão* e acabarão "sufocando" e quebrando a porta de entrada do servidor de dados, impossibilitando qualquer outra API de se conectar nas próximas semanas.
</details>




