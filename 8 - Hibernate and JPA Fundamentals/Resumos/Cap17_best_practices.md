# Capítulo 17: Melhores Práticas (Best Practices)

Neste capítulo final, consolidamos todas as regras de ouro e melhores práticas de performance, escalabilidade e arquitetura que foram aprendidas ao longo do curso. Estas são as diretrizes que devem guiar um projeto JPA/Hibernate no mundo real.

## 1. Identificadores (Chaves Primárias)
- **Regra do Significado de Negócio:** Seu atributo `@Id` **NUNCA** deve ter significado de negócio. Ele deve ser apenas um identificador de banco de dados (Surrogate Key). Chaves naturais (como CPF ou Matrícula) não devem ser o `@Id`.
- **Sempre prefira `GenerationType.SEQUENCE`:** Se o seu banco de dados (ex: PostgreSQL ou Oracle) tiver suporte nativo a Sequences, use. Ela é uma estratégia *Pre-INSERT*, o que significa que o Hibernate consegue descobrir os IDs com antecedência e isso habilita o **JDBC Batching** (agrupamento de inserts em lote).
- **Evite a estratégia `TABLE`:** Em aplicações multiusuário com grande volume de inserts concorrentes, a estratégia `TABLE` sofre problemas crônicos de escalabilidade, pois ela usa travamentos (locks) físicos no banco para garantir que dois objetos não peguem o mesmo ID.
- **A Exceção MySQL:** O MySQL **não** tem suporte nativo a sequences. Se você usar `SEQUENCE` nele, o Hibernate silenciosamente fará um *fallback* para a péssima estratégia `TABLE`. No MySQL, use SEMPRE `GenerationType.IDENTITY`.

## 2. O Método Equals e HashCode
Sempre implemente os métodos `equals()` e `hashCode()` baseando-se nas **Chaves Naturais** (Business Keys) da sua Entidade, e não no ID de banco gerado.
- Isso é estritamente obrigatório quando você vai trabalhar com **Objetos Destacados (Detached)**, especialmente se for colocá-los dentro de um `Set` ou usá-los como chaves em um `Map`. O Java confia cegamente no `hashCode/equals` para impedir duplicidades nesses conjuntos.

## 3. Mapeamento de Associações
- **Prefira Associações Bidirecionais:** Associações unidirecionais engessam o seu modelo. Na vida real, relatórios e telas costumam exigir navegação nas duas direções da associação.
- **Sempre prefira o Fetching LAZY:** 
  - Coleções (`@OneToMany`, `@ManyToMany`) já são Lazy por padrão.
  - Mas os tipos Singulares (`@ManyToOne`, `@OneToOne`) são **EAGER por padrão**. Você **deve** mudá-los explicitamente para `(fetch = FetchType.LAZY)`. Carregar dados extras que a tela não pediu destrói o tráfego de rede e a memória.

## 4. Otimização do Cache de Primeiro Nível (L1) e Dirty Checking
Mantenha o tamanho do seu Contexto de Persistência (`EntityManager`) o menor possível.
- **Motivo:** O Hibernate, ao carregar um objeto, tira uma foto idêntica dele em memória (chamada de Snapshot). No final da transação, ele faz o **Dirty Checking**, comparando o objeto atual contra sua foto para descobrir se precisa fazer um `UPDATE`. Se o seu L1 tem 10.000 objetos desnecessários, o Hibernate demorará uma eternidade fazendo milhares de comparações só para chegar à conclusão de que nada mudou. Só carregue o que você realmente precisa modificar.

## 5. Consultas e Segurança
Sempre utilize **Bind Variables** (Parâmetros Nomeados, ex: `:nome_variavel`) ao fazer consultas dinâmicas JPQL, HQL ou JDBC puro. 
- Jamais use concatenação de Strings (`"select * from T onde id = " + variavel`). Isso não só abre as portas para SQL Injection como destrói a otimização de plano de execução (*Execution Plan*) no cache interno do banco de dados.

## 6. Processamento em Lote (Batch Processing)
- Ao processar milhares de atualizações, inserções ou deleções, **sempre use Batch Processing** (configurando `hibernate.jdbc.batch_size`).
- Lembre-se de fazer `em.flush()` e `em.clear()` a cada iteração do lote (ex: a cada 50 objetos). Se não fizer isso, a memória da JVM estourará (*Out of Memory*).
- **Aviso Crucial:** O Hibernate é fisicamente incapaz de fazer JDBC Batching de `INSERTS` se a sua entidade usar a estratégia `GenerationType.IDENTITY`. O `IDENTITY` exige a execução imediata do SQL para descobrir o ID gerado pelo banco.

## 7. O Problema do N+1 Selects
O fetching `LAZY` é a regra de ouro, mas iterar num `for-loop` sobre proxies Lazy dispara a famosa metralhadora de Selects (N+1 Problema).
- **A Solução:** Quando souber de antemão que vai precisar dos filhos na tela/retorno da API, substitua a requisição normal por uma consulta JPQL explicitando `JOIN FETCH` (ou usando `@NamedEntityGraph`). Isso carrega os dados em uma única viagem ao banco de dados, contornando cirurgicamente a barreira Lazy.

## 8. Evite "Atualizações Perdidas" (Lost Updates)
Numa aplicação corporativa onde os usuários mantêm "Conversações longas" e concorrentes (onde carregam objetos para uma interface, pensam durante minutos, e clicam em Salvar via estado Destacado), ocorrem "Atualizações Perdidas" baseadas na regra de que o *Último Commit Vence*.
- **Prevenção:** Sempre implemente o **Optimistic Locking**, criando uma coluna de versão mapeada com a anotação `@Version`. Se houver choque temporal, o sistema rejeitará o último salvamento lançando a exceção `OptimisticLockException`.

## 9. Caching de Segundo Nível (L2)
O uso do Caching L2 **não é** para todo mundo. Ative-o unicamente se:
1. Os dados mudarem muito pouco na tabela.
2. A leitura for incrivelmente superior à escrita (ex: Países, Categorias, Configurações Base do Sistema).
3. O banco de dados **NÃO** for modificado/escrito por outras aplicações externas que o Hibernate desconheça (o Hibernate não sabe quando outra linguagem alterou o banco, então os dados do L2 ficarão zumbis/obsoletos).
