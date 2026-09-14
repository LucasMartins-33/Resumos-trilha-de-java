# Questões Práticas - Capítulo 24 (Java Database Connectivity - JDBC)

🟢 Nível 1: O Setup (Adicionando o Driver)
Cenário: Você não encontrou a classe Driver.
Sua Tarefa (Teórica de fluxo):
* Descreva em passos rápidos (ou apenas comente mentalmente) o que deve ser feito com o arquivo `.jar` que a equipe de banco de dados te entregou para que os erros sumam do Eclipse. (R: Colar o jar na pasta do projeto e ir no `Java Build Path > Libraries > Add JARs`).

🟡 Nível 2: A Ponte (A Connection String)
Cenário: O banco de dados roda na máquina local na porta padrão, e se chama "controle_escola".
Sua Tarefa:
* Crie a variável String contendo a URL exata do banco usando a arquitetura oficial do MYSQL JDBC.
* (Dica: `jdbc:mysql://[host]:[porta]/[banco]`).

🟠 Nível 3: O Teste de Acesso
Cenário: A string de conexão está pronta. Vamos testar a entrada.
Sua Tarefa:
* Envolva os códigos do nível anterior num bloco `try / catch(SQLException e)`.
* Dentro do try, obtenha uma conexão (A variável Connection do pacote `java.sql`).
* Invoque estaticamente a classe Driver: `DriverManager.getConnection(url, "login", "senha");`

🔴 Nível 4: Preparando a Encomenda (Statement)
Cenário: A porta do banco está aberta e precisamos enviar um pacote de busca de informações.
Sua Tarefa:
* Com a variável `conn` de Connection estabelecida e funcional no nível anterior, extraia um carteiro.
* Inicialize um objeto `Statement stmt = conn.createStatement();`.
* *Lembrete:* Statement, ResultSet e Connection sempre devem ser do pacote `java.sql.*`! Cuidado com os auto-imports da IDE que podem trazer classes erradas de outros pacotes aleatórios.

🟣 Nível 5: O Saque (SELECT e ResultSet)
Cenário: Queremos ver quem são os alunos cadastrados!
Sua Tarefa:
* Com o Statement criado, execute a String `SELECT * FROM alunos`.
* Sabendo que a operação é apenas buscar uma tabela para o Java, invoque o `stmt.executeQuery("...")`.
* Intercepte a tabela de retorno inteira criando na ponta esquerda e declarando uma variável do tipo `ResultSet rs = ...` para receber a pancada do resultado.

🟤 Nível 6: Varrendo e Deserializando a Tabela
Cenário: O pacote veio fechado. O `ResultSet` contém dados densos e precisamos convertê-los em algo usável para o print da tela do Java.
Sua Tarefa:
* Faça um `while (rs.next()) { }` logo em seguida.
* Sabendo que o banco de dados tem a coluna "nome" do tipo VARCHAR e a coluna "idade" do tipo numérico, declare dentro das chaves duas variáveis extraindo esses dados e forçando uma tipagem sólida no Java (Use o `.getString("nome")` e o `.getInt("idade")` atrelados ao objeto `rs`). Imprima-as no console.

🔵 Nível 7: Modificando a Base (INSERT)
Cenário: Tem aluno novo na área (João, Idade 14).
Sua Tarefa:
* A operação agora muda radicalmente. Construa uma String contendo a instrução em SQL puro: `INSERT INTO alunos (nome, idade) VALUES ('João', 14)`.
* Sabendo que não estamos mais lendo tabelas, mude a execução do Statement. Em vez de Query, execute um `stmt.executeUpdate(stringDeInsert)`.

🟢 Nível 8: O Confirmador de Linhas (Rows Affected)
Cenário: Em atualizações em massa de sistemas, o diretor de dados quer o log que certifique para o sistema do Java o impacto destrutivo exato que o seu último comando gerou na base.
Sua Tarefa:
* Lembre-se que todo e qualquer comando de `executeUpdate()` invocado num statement nunca devolve lixo ou nulo, mas devolve e cospe especificamente um inteiro matemático.
* Intercepte e declare esse `int linhasAfetadas = stmt.executeUpdate(...)` do Nível 7, e apenas printe essa variável na tela para fins de log de console com sucesso na alteração.

🟡 Nível 9: A Barreira contra Desastres (UPDATE e WHERE)
Cenário: O professor mandou você aumentar a nota do João (que tem ID = 5) de 5.0 para 9.0. Você escreveu `UPDATE alunos SET nota = 9.0;`. Você acaba de dar nota 9.0 para **todos** os alunos da escola. O projeto ruiu. 
Sua Tarefa (Correção):
* Reescreva a String isolada do código acima com o uso crucial da barreira lógica impeditiva do SQL puro.

🟠 Nível 10: Limpeza de Recursos Arquiteturais
Cenário: Você escreveu os scripts e foi embora. Uma semana depois, o servidor do banco quebrou por invasões de recursos bloqueados por conexões mortas ("Connections Leak").
Sua Tarefa:
* Utilize a mecânica moderna do `Try-with-Resources` introduzida no capítulo passado e integre-a magistralmente ao JDBC.
* Em vez de declarar a `Connection`, o `Statement` e o `ResultSet` em linhas soltas no corpo da função de forma vulnerável, mude o design e os declare perfeitamente empilhados com quebras de linhas **dentro dos parênteses curvos `()` do próprio block try**, garantindo proteção e imunidade contra fechamentos corrompidos.
