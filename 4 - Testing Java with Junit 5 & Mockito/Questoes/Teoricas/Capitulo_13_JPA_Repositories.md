# Questões Teóricas - Capítulo 13: Testando JPA Repositories

Testes de fixação sobre o que testar em Spring Data JPA, Query Methods derivados, consultas customizadas com `@Query` e isolamento da camada de persistência.

---

### 1. Qual é a "Regra de Ouro" sobre o que NÃO se deve testar ao criar testes automatizados para interfaces de repositório do Spring Data JPA?

<details>
<summary>👀 Ver Resposta</summary>

Não se deve gastar tempo escrevendo testes automatizados para os métodos básicos pré-fabricados herdados diretamente de interfaces como `CrudRepository`, `ListCrudRepository` ou `JpaRepository` (por exemplo: `save()`, `findById()`, `findAll()`, `delete()`, `count()`). Essas rotinas já foram exaustivamente testadas pela equipe mantenedora do ecossistema Spring Data. Criar testes repetitivos para essas chamadas gera custos de manutenção desnecessários sem agregar valor real ao negócio.
</details>

---

### 2. O que realmente DEVE ser testado em uma interface que herda de `JpaRepository`?

<details>
<summary>👀 Ver Resposta</summary>

Deve-se testar estritamente o código e as regras customizadas adicionadas pela equipe do projeto:
1. **Query Methods (Consultas derivadas por nome):** Métodos cuja consulta SQL é gerada a partir da interpretação do nome do método (ex: `findByEmailAndFirstName`).
2. **Consultas customizadas com a anotação `@Query`:** Métodos que utilizam consultas explícitas escritas pelo desenvolvedor em JPQL ou SQL Nativo.
</details>

---

### 3. Por que Query Methods derivados do nome (ex: `findByEmailAndStatus`) são alvos críticos de testes automatizados?

<details>
<summary>👀 Ver Resposta</summary>

Porque o Spring Data interpreta dinamicamente as palavras-chave do nome do método para montar a árvore de predicados e gerar as instruções SQL correspondentes. Qualquer erro ortográfico sutil, confusão entre operadores (`And` vs `Or`) ou troca de ordem de propriedades no nome do método pode alterar silenciosamente o filtro da consulta ou causar falhas na inicialização do repositório, exigindo testes que certifiquem que a query produz exatamente o conjunto de dados esperado.
</details>

---

### 4. Qual é o risco associado ao uso de consultas manuais declaradas com `@Query` (seja JPQL ou SQL Nativo) e por que os testes são obrigatórios?

<details>
<summary>👀 Ver Resposta</summary>

Quando o desenvolvedor escreve manualmente uma string com JPQL ou SQL Nativo dentro de `@Query("SELECT u FROM User u WHERE...")`, ele assume toda a responsabilidade pela sintaxe, joins, nomes de colunas e cláusulas lógicas. O compilador Java não valida a semântica da string em tempo de compilação. Testes automatizados executam essa consulta contra o mecanismo de persistência, comprovando que a consulta compila perfeitamente e filtra as linhas com precisão matemática.
</details>

---

### 5. Em um teste de repositório anotado com `@DataJpaTest`, qual é o papel do `TestEntityManager` na etapa Arrange?

<details>
<summary>👀 Ver Resposta</summary>

O `TestEntityManager` é a ferramenta responsável por criar e persistir fisicamente no banco de dados a massa de dados preparatória (entidades cobaia). Utilizar o `TestEntityManager` em vez de chamar `repository.save()` na etapa Arrange desacopla o teste: garante que a preparação dos dados dependa diretamente do contexto de persistência padrão do JPA e não da própria interface de repositório que está sob avaliação.
</details>

---

### 6. Por que o método `persistAndFlush()` é fundamental ao preparar cenários de teste para repositórios?

<details>
<summary>👀 Ver Resposta</summary>

Porque as consultas customizadas do repositório (especialmente queries derivadas e JPQL) realizam buscas diretamente no banco de dados relacional. Se os dados da etapa Arrange fossem apenas mantidos no cache em memória da sessão sem um `flush`, a consulta disparada pelo repositório poderia não enxergar os novos registros na tabela física, gerando resultados vazios e falsos negativos nos testes.
</details>

---

### 7. Por que, ao testar uma consulta de repositório que retorna uma lista filtrada, é considerado essencial popular a tabela com dados "mistos"?

<details>
<summary>👀 Ver Resposta</summary>

Popular a tabela apenas com registros que atendem à condição do filtro não prova que o filtro funciona de verdade (uma consulta defeituosa que retorne a tabela inteira passaria no teste). É indispensável inserir dados que **atendem** ao critério e dados que **não atendem** (cenários positivos e negativos simultâneos) e certificar que apenas os itens corretos foram retornados pela consulta (`assertEquals(1, lista.size())`), validando o poder de restrição do SQL.
</details>

---

### 8. Como deve ser validada uma consulta de busca pontual de usuário por e-mail (`findByEmail`)?

<details>
<summary>👀 Ver Resposta</summary>

1. **Arrange:** Cria uma entidade de usuário com um e-mail específico (ex: `lucas@teste.com`), persiste e força a sincronização física com `testEntityManager.persistAndFlush(user)`.
2. **Act:** Invoca o método sob teste na interface do repositório: `UserEntity result = repository.findByEmail("lucas@teste.com");`.
3. **Assert:** Valida primeiramente a existência do objeto (`assertNotNull(result)`) e, em seguida, assere que as propriedades coincidem (`assertEquals("lucas@teste.com", result.getEmail())`).
</details>

---

### 9. O que ocorre com os registros persistidos durante a execução de um teste de repositório sob `@DataJpaTest` ao término do método de teste?

<details>
<summary>👀 Ver Resposta</summary>

Graças ao comportamento transacional padrão do `@DataJpaTest`, todo método de teste é executado dentro de uma transação isolada que sofre **rollback automático** imediatamente após sua conclusão. Portanto, todos os registros criados pelo `testEntityManager` são sumariamente descartados, garantindo que as tabelas voltem ao estado original sem poluir os testes subsequentes.
</details>

---

### 10. Por que testes de repositório com `@DataJpaTest` não devem injetar classes da camada de Serviço (`@Service`) ou de Controle (`@Controller`)?

<details>
<summary>👀 Ver Resposta</summary>

Porque a premissa fundamental de um teste de persistência é a verificação isolada da interação com a base de dados. Incluir classes de serviço ou controle aumenta o acoplamento, traz dependências externas indesejadas, torna a inicialização do contexto do Spring substancialmente mais lenta e mascara o diagnóstico: se o teste falhar, fica difícil discernir se o defeito residia na lógica de negócios do serviço ou na construção da query SQL do repositório.
</details>
