# Capítulo 05: Control Structures & Errors (Estruturas de Controle e Erros)

Estruturas de controle (Ifs, For loops) são os maiores causadores de código ilegível quando mal utilizados. O principal inimigo aqui é o **"Arrow Code" (Código em forma de flecha)**, causado por um aninhamento profundo de `ifs` dentro de loops dentro de outros `ifs`.

Neste capítulo, aprendemos técnicas pesadas de refatoração para transformar um código aninhado e confuso em algo linear e limpo.

## 1. O Poder dos Guards (Fail Fast)
A forma mais rápida de remover níveis de aninhamento (nesting) é usando **Guards** (Guardas). Em vez de checar a condição ideal e colocar o corpo da função inteira dentro do bloco `if`, você **inverte a lógica**, checa o cenário de falha e faz um retorno antecipado (Fail Fast).

```javascript
// 🔴 Ruim (Nesting desnecessário)
function process(email) {
    if (email.includes('@')) {
        // ... 50 linhas de código de processamento
    }
}

// 🟢 Clean (Usando um Guard)
function process(email) {
    if (!email.includes('@')) return; // Fail fast
    
    // ... 50 linhas de código soltas, sem aninhamento
}
```
*Dica:* Em loops (`for`/`while`), os guards utilizam a palavra-chave `continue` em vez de `return` para pular a iteração defeituosa e seguir para a próxima.

## 2. Refatorando Expressões e Estruturas
- **Prefira Fraseado Positivo:** Ao criar variáveis ou funções booleanas, prefira sentenças positivas, pois são mais naturais para o cérebro processar. Use `isEmpty()` em vez de `hasNoElements()`.
- **Extraia Condicionais:** Condições complexas como `if (transaction.type === 'PAYMENT' && transaction.status === 'OPEN')` devem ser extraídas para funções com nomes semânticos, como `if (isValidPayment(transaction))`.

## 3. Abrace os Erros (Error Handling)
Não tenha medo de lançar erros reais (`throw new Error()`). Muitos desenvolvedores tentam usar `Ifs` e retornar objetos falsos de erro (ex: `return { code: 422, msg: 'Failed' }`). Isso não é "Clean"! 
- **Use as ferramentas da linguagem:** Se ocorreu uma falha real na validação de um dado, lance um erro (Throw) e deixe ele "borbulhar" até o nível certo de interceptação (`try/catch`).
- **Tratamento de Erro é "Uma Coisa" (One Thing):** Se uma função faz validações complexas, tem loops extensos e no meio disso tudo um grande `try/catch`, ela está fazendo coisas demais. Idealmente, o `try/catch` deve delegar o corpo da tentativa (`try`) para outra função atômica.

## 4. Factory Functions para evitar Duplicação de Ifs
Quando você se pegar repetindo blocos enormes de `if / else if / else` para rotear funções baseadas num tipo (ex: pagamentos com Cartão, PayPal ou Boleto), utilize **Factory Functions** em conjunto com Dicionários/Mapas de funções.

Em vez de rotear o código de forma imperativa:
```javascript
// 🔴 Ruim: Muito Boilerplate e repetição
if (method === 'CREDIT_CARD') {
    if (type === 'PAYMENT') processCreditCardPayment();
    else processCreditCardRefund();
}
```

Retorne um objeto mapeado com referências de funções (que não são executadas imediatamente, apenas passadas por referência):
```javascript
// 🟢 Clean: Retornando um "Dicionário" de roteamento
function getProcessors(method) {
    if (method === 'CREDIT_CARD') {
        return { 
            payment: processCreditCardPayment, 
            refund: processCreditCardRefund 
        };
    }
    // ...
}

// Chamando a factory:
const processors = getProcessors(tx.method);
processors.payment(tx); // Executa a função correta
```
*Nota: Este é um princípio inicial de Polimorfismo. O curso aprofundará nisso no capítulo de Classes/Objetos.*

## 5. Micro-otimizações de Limpeza
- **Parâmetros Padrão (Default Parameters):** Evite `ifs` checando por `undefined` setando parâmetros diretamente na assinatura da função: `function printError(msg, item = {})`.
- **Evite Números e Strings Mágicos:** Não espalhe textos hard-coded (ex: `'CREDIT_CARD'`) pelo seu código. Use constantes ou Enums globais (ex: `const TYPE_CREDIT_CARD = 'CREDIT_CARD'`). Isso evita erros de digitação e facilita refatorações em massa no futuro.
