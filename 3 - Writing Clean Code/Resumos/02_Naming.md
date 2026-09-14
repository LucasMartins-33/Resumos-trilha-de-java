# Capítulo 02: Naming (Nomenclatura) - Variáveis, Funções e Classes

## 1. A Regra Geral: Nomes Significativos
A regra principal para atribuir nomes no código é simples: **nomes devem ter significado**. Ao ler o nome de uma variável, função ou classe, você deve conseguir entender o que ela armazena ou faz sem precisar analisar a implementação ou o valor que está sendo passado a ela. 

## 2. Padrões de Escrita (Casing Conventions)
As convenções não mudam a regra do Clean Code, mas fazem parte da organização do projeto e dependem da linguagem. As mais comuns são:
- **`snake_case`:** Tudo minúsculo com palavras separadas por underscore. Comum no Python para variáveis, funções e métodos (ex: `is_valid`, `send_response`).
- **`camelCase`:** A primeira letra é minúscula e cada nova palavra começa com maiúscula. Muito usado em JavaScript/TypeScript e Java para variáveis, métodos e funções (ex: `isValid`, `sendResponse`).
- **`PascalCase`:** Todas as palavras começam com letra maiúscula. Quase universalmente utilizado para nomes de **Classes** (ex: `User`, `DatabaseManager`).
- **`kebab-case`:** Minúsculas separadas por traços. Comum em HTML/CSS e URLs (ex: `my-custom-element`).

## 3. Como Nomear Variáveis e Propriedades
Variáveis e propriedades armazenam dados. Logo, o nome deve **descrever o conteúdo**.

### Valores Gerais (Strings, Números, Objetos)
Use **substantivos** ou frases curtas focadas em substantivos.
- 🔴 **Ruim:** `u`, `data`, `val`, `n`
- 🟡 **Ok:** `userData`, `person` (podem ser genéricos demais)
- 🟢 **Bom:** `user`, `customer`, `firstName`, `admin`

### Valores Booleanos (Verdadeiro ou Falso)
Nomes de variáveis booleanas devem ser formulados como uma **pergunta que pode ser respondida com sim ou não** (`true` ou `false`).
- 🔴 **Ruim:** `correct`, `validatedInput`
- 🟢 **Bom:** `isActive`, `isLoggedIn`, `isValid`, `hasError`

## 4. Como Nomear Funções e Métodos
Funções executam lógicas ou calculam algo. O nome deve refletir um **comando** ou uma **ação**.

### Operações Gerais
Use **verbos** associados a um contexto de ação.
- 🔴 **Ruim:** `process()`, `handle()` (muito vagos, não dizem qual é a ação real)
- 🟡 **Ok:** `save()`, `storeData()`
- 🟢 **Bom:** `saveUser()`, `getUserByEmail()`, `printBlogPost()`

### Retorno Booleano
Se a função simplesmente avalia uma condição e retorna um booleano, ela pode usar a mesma convenção de variáveis booleanas (exceção à regra dos verbos):
- 🟢 **Bom:** `emailIsValid()`, `isPaid()`

## 5. Como Nomear Classes
Classes são moldes usados para **criar objetos**. Portanto, o nome deve refletir o objeto que será instanciado.
- Use **Substantivos**.
- 🔴 **Ruim:** `UEntity`, `ObjA`, `DatabaseManager` (a menos que seja apenas um pacote estático de funções utilitárias)
- 🟡 **Ok:** `UserObj`, `AppUser` (são redundantes)
- 🟢 **Bom:** `User`, `Admin`, `Customer`, `SQLDatabase`

## 6. Exceções e Cuidados
- **Getters e Setters:** Em muitas linguagens (como JS/TS), `getters` funcionam como métodos por baixo dos panos, mas são acessados como propriedades. Por isso, eles devem seguir a convenção de nome de *variáveis* (ex: `connectedClient` ao invés de `getConnectedClient`).
- **Bibliotecas Padrão:** Muitas libs antigas possuem nomes péssimos (como o `datetime.now()` e `strftime` no Python, que não parecem funções/métodos pelas regras tradicionais). Você não precisa copiar essas más práticas para os seus métodos customizados.
- **Classes Utilitárias (Utils):** Sufixos como `Util` ou `Manager` só são aceitáveis para classes que agrupam funções estáticas que não representam um objeto do mundo real (ex: `DateUtil`).

## 7. Erros Comuns e Armadilhas (Pitfalls)
1. **Informação Redundante:** Não inclua informações no nome que já são óbvias pelo contexto.
   - 🔴 `userWithNameAndAge` ➔ 🟢 `user`
   - 🔴 `class Point { constructor(coordX, coordY) }` ➔ 🟢 `class Point { constructor(x, y) }`
2. **Uso de Gírias ou Termos Pouco Claros:**
   - 🔴 `diePlease()` ➔ 🟢 `remove()` ou `delete()`
   - 🔴 `build_stuff()` ➔ 🟢 `build_rectangle()`
3. **Desinformação (Misleading Names):**
   - 🔴 `userList` (quando na verdade o dado é um Dicionário/Objeto e não um Array/Lista).
   - 🔴 `allAccounts` (quando o array na verdade contém apenas contas filtradas e pagas; melhor chamar de `paidAccounts`).
4. **Falta de Consistência:** Se você escolheu o prefixo `get` para buscar dados (`getUsers()`), não use `fetch` ou `retrieve` em outras partes do código para fazer a mesma coisa (`fetchProducts()`). Mantenha um único padrão (ou tudo `get`, ou tudo `fetch`).

## 8. Casos Práticos: Antes e Depois (Refatorações do Curso)

### Caso 1: Refatorando Função para Método de Classe
*Dica:* Em vez de criar um nome de função excessivamente longo para compensar dados externos, traga a função para dentro da classe do objeto que ela manipula.

**🔴 Antes (Procedural e Nomes Ruins):**
```python
# Nomes péssimos: Entity, ymdhm (data), output
class Entity:
    def __init__(self, title, description, ymdhm):
        self.title = title
        self.description = description
        self.ymdhm = ymdhm

def output(item):
    print(item.title, item.description, item.ymdhm)
```

**🟢 Depois (Clean Code):**
```python
# Nomes claros, e a função virou método da classe, removendo a necessidade
# de passar um argumento extra e usar verbos compostos gigantes.
class BlogPost:
    def __init__(self, title, description, date_published):
        self.title = title
        self.description = description
        self.date_published = date_published

    def print(self):
        print(self.title, self.description, self.date_published)

# Uso muito mais claro:
post = BlogPost("Clean Code", "Summary", "2024-01-01")
post.print()
```

### Caso 2: Cuidado com Medidas e Posições
**🔴 Antes (Vocabulário estranho):**
```python
class Rectangle:
    def __init__(self, starting_point, broad, high):
        self.starting_point = starting_point
        self.broad = broad # Gíria/Inglês estranho
        self.high = high

    def area(self): # Parece nome de variável, mas é função matemática
        return self.broad * self.high
```

**🟢 Depois (Convenções e Nomes precisos):**
```python
class Rectangle:
    def __init__(self, origin, width, height):
        self.origin = origin
        self.width = width
        self.height = height

    def get_area(self): # Fica explícito que é um comando/cálculo
        return self.width * self.height
```
