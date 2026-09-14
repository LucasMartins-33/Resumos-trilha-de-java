# Capítulo 24: JDBC (Java Database Connectivity)

Este capítulo introduz o **JDBC** (Java Database Connectivity), que é a API nativa do Java usada para comunicar aplicações Java com bancos de dados relacionais (como MySQL, Oracle, SQL Server, etc.).

## 1. Conceitos Básicos e Setup

*   **Database (Banco de Dados):** Software utilizado para armazenar e gerenciar dados de forma estruturada (semelhante a planilhas interligadas).
*   **JDBC API:** Conjunto de bibliotecas embutidas no Java (`java.sql.*`) usadas para interagir com o banco de dados.
*   **Driver do Banco de Dados (Vendor Driver):** É o "tradutor" ou intermediário. Como cada banco de dados funciona de forma diferente por debaixo dos panos, as empresas (Oracle, Microsoft, etc.) fornecem um arquivo `.jar` (Driver) específico para que o JDBC do Java consiga conversar com aquele banco em particular.

**Passo a passo no Eclipse (Setup do MySQL):**
1. Instalar o MySQL Server e o MySQL Workbench (Interface gráfica para gerenciar o banco).
2. Baixar o arquivo `.jar` do "MySQL Connector/J" (o driver JDBC do MySQL).
3. No Eclipse: Criar uma pasta `lib` no projeto, colar o `.jar` dentro dela.
4. Clicar com botão direito no projeto -> `Properties` -> `Java Build Path` -> `Libraries` -> `Add JARs...` e selecionar o driver. Sem isso, o Java lançará erro dizendo que não encontrou o driver adequado.

## 2. Conectando e Lendo Dados (SELECT)

Para buscar dados de uma tabela e usá-los no Java, seguimos esta estrutura de objetos da biblioteca `java.sql`:

1.  **Connection:** Representa a conexão aberta com o banco.
2.  **Statement:** O objeto que carregará e executará a sua "pergunta" (Query) em SQL.
3.  **ResultSet:** O objeto que recebe e armazena a tabela de resultados devolvida pelo banco.

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.Statement;
import java.sql.SQLException;

public class JDBCDemo {
    public static void main(String[] args) {
        
        // 1. Definir a URL de conexão (Formato padrão do JDBC para MySQL)
        // Sintaxe: jdbc:mysql://[servidor]:[porta]/[nome_do_banco]
        String url = "jdbc:mysql://127.0.0.1:3306/employees_database";
        
        try {
            // 2. Estabelecer a Conexão
            Connection conn = DriverManager.getConnection(url, "root", "password123");
            
            // 3. Criar o Statement
            Statement statement = conn.createStatement();
            
            // 4. Executar a Query SQL (O executeQuery é usado para consultas do tipo SELECT)
            ResultSet resultSet = statement.executeQuery("SELECT * FROM employees_tbl");
            
            // 5. Processar o ResultSet (O .next() itera linha por linha nos resultados)
            while (resultSet.next()) {
                // Podemos pegar o dado como String
                String nome = resultSet.getString("name");
                
                // Ou podemos pegar o dado no seu tipo primitivo direto (ex: INT para cálculos)
                int salario = resultSet.getInt("salary");
                
                System.out.println(nome + " ganha $" + salario);
            }
            
        } catch (SQLException e) {
            System.out.println("Erro de conexão ou sintaxe SQL!");
            e.printStackTrace();
        }
    }
}
```

## 3. Alterando Dados (INSERT, UPDATE, DELETE)

Quando não queremos *ler* dados, mas sim *modificar* o banco (inserir um novo registro, deletar ou atualizar algo), **NÃO usamos** o método `.executeQuery()`.

Para modificar dados, usamos o método **`.executeUpdate()`**.
*   Ao invés de retornar um `ResultSet` (tabela), ele retorna um número `int` (quantidade de linhas afetadas no banco).

```java
// O setup de Connection e Statement é idêntico ao exemplo acima...

try {
    Connection conn = DriverManager.getConnection(url, "root", "password123");
    Statement statement = conn.createStatement();
    
    // --- EXEMPLO DE INSERT ---
    String sqlInsert = "INSERT INTO employees_tbl (id, name, dept, salary) VALUES (900, 'Robert', 'Sales', 4000)";
    
    // --- EXEMPLO DE UPDATE ---
    // Sempre use WHERE em UPDATES para não alterar a tabela inteira por acidente!
    String sqlUpdate = "UPDATE employees_tbl SET salary = 5500 WHERE id = 600";
    
    // --- EXEMPLO DE DELETE ---
    // Sempre use WHERE em DELETES para não apagar a tabela inteira por acidente!
    String sqlDelete = "DELETE FROM employees_tbl WHERE id = 900";
    
    
    // EXECUTANDO UM COMANDO (Usando o UPDATE como exemplo)
    int rowsAffected = statement.executeUpdate(sqlUpdate);
    
    System.out.println("Comando executado com sucesso! Linhas alteradas: " + rowsAffected);
    
} catch (SQLException e) {
    System.out.println("Erro na alteração dos dados!");
    e.printStackTrace();
}
```

### Resumo das Regras:
*   Para **Consultas (`SELECT`)**: Use `statement.executeQuery(sql)` => Retorna um `ResultSet`.
*   Para **Modificações (`INSERT`, `UPDATE`, `DELETE`)**: Use `statement.executeUpdate(sql)` => Retorna um `int` (Rows Affected).
