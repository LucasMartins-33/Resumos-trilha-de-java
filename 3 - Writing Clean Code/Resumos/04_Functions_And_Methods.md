# Capítulo 04: Functions & Methods (Funções e Métodos)

As funções/métodos abrigam a maior parte do código que escrevemos. Criar funções limpas envolve lidar não apenas com a complexidade do que está "dentro" delas, mas também com a forma que as chamamos.

## 1. Parâmetros e Argumentos
A regra de ouro é: **Minimize o número de parâmetros**. Quanto mais argumentos uma função recebe, mais difícil é chamá-la e mais provável é de você (ou um colega) esquecer ou errar a ordem deles.

- **0 parâmetros (Excelente):** Funções fáceis de ler e chamar (ex: `user.save()`).
- **1 a 2 parâmetros (Bom/Aceitável):** Totalmente válidos desde que a ordem seja óbvia e intuitiva (ex: `Point(x, y)`).
- **3 parâmetros (O Limite):** Evite ao máximo. A ordem muitas vezes se perde.
- **>3 parâmetros (Ruim):** Código hostil à manutenção. Refatore.

### Como lidar com muitos parâmetros?
Quando você realmente precisa de 4, 5 ou mais pedaços de dados, **agrupe-os em um único objeto ou dicionário**. Assim, você passa **apenas 1 parâmetro** e a ordem para de importar porque os dados são acessados por chaves (keys).

*Exemplo (JS/TS):*
```javascript
// 🔴 Ruim: Qual é a ordem mesmo? 
const u = new User('max@test.com', 31, 'Max'); 

// 🟢 Bom: Passando um Objeto. Ordem irrelevante, alta clareza!
const u = new User({ 
    name: 'Max',
    email: 'max@test.com',
    age: 31
});
```

### O caso especial: Parâmetros Dinâmicos (Rest/Spread)
O uso de um número dinâmico de parâmetros (`...args` em JavaScript, ou `*args` em Python) é uma **exceção válida** para a regra, pois, no fim, você está apenas empacotando os argumentos num único Array e, comumente, executando a mesma lógica (ex: `sumAll(1, 2, 3, 4, 5)`).

### Output Parameters (Evite!)
Um parâmetro de saída (Output Parameter) ocorre quando você altera o objeto que foi passado **silenciosamente**.
```javascript
// 🔴 Ruim: Altera o objeto 'user' sem deixar isso explícito no nome.
function createId(user) {
    user.id = generateHash();
}

// 🟢 Bom (Pelo menos esperado): O nome adverte sobre a alteração
function addId(user) { ... }

// 🟢 Excelente (Abordagem Orientada a Objeto)
user.addId();
```

---

## 2. O Corpo da Função
Funções devem ser **pequenas e fazer apenas UMA coisa** (Do One Thing). 

Mas o que é "uma coisa"? 
- Para medir "uma coisa", olhe para os **Níveis de Abstração**. O nome da função é o "nível mais alto". Tudo que está dentro dela deveria estar exatamente no degrau abaixo do nome, orquestrando passos lógicos. 
- Se você tem uma função `saveUser()` e dentro dela há código checando `if (email.includes('@'))`, você está quebrando os níveis. A checagem de caracteres na string é um "baixo nível" (API da linguagem), enquanto `saveUser` é "alto nível" (Regra de negócio). O correto seria orquestrar:
  ```javascript
  // 🟢 Clean
  function saveUser(data) {
      validateInput(data); // Alto nível (esconde os low-levels dentro dela)
      db.insert(data);
  }
  ```

### DRY (Don't Repeat Yourself)
Se você se flagrar copiando lógicas (como validar um e-mail em dois lugares do sistema), você quebrou o princípio DRY. Extraia a lógica copiada para uma função independente e chame-a nos dois locais originais.

### Não exagere nas extrações
Se você precisar dividir uma função apenas para cumprir tabela, e o nome da nova função for um **sinônimo perfeito** da antiga (ex: a função nova se chama `buildUser` dentro de uma função `createUser`), você provavelmente fez uma "Extração Inútil". Não fragmente o código sem motivos.

---

## 3. Side Effects (Efeitos Colaterais) e Funções Puras
- **Função Pura (Pure Function):** Para os mesmos parâmetros (inputs), **sempre** retornará o mesmo resultado. E principalmente: ela **não afeta nada** fora do escopo dela.
- **Side Effect:** Tudo aquilo que altera o estado do programa/mundo externo. Escrever um arquivo, enviar uma request HTTP, modificar o Banco de Dados, atualizar uma global ou um `console.log()` na tela.

**O Problema não é ter Efeitos Colaterais, mas eles serem Inesperados.**
Nomes enganosos causam Side Effects inesperados. Se uma função se chama `isValid(user)`, seu único trabalho é retornar `true` ou `false`. Se, no meio da validação, ela mostra um Alert na tela ou envia um log no Banco de Dados, este efeito é inesperado. Código limpo faz apenas o que seu nome propõe! 

---

## 4. Por que Testes Unitários importam para Código Limpo?
O ato de tentar escrever **Testes Unitários** é a prova de fogo de um código limpo.
- Se uma função é difícil de testar, requer `mocks` difíceis ou testa mil cenários completamente diferentes, isso quer dizer que: **sua função é grande demais, ou faz coisas demais (muitos Side Effects indesejáveis em uma única caixa)**.
- Se a sua base é construída por funções pequenas, puras e focadas em uma única coisa, cobri-las de testes demora minutos e protege sua aplicação inteira (TDD / Test Driven Development).
