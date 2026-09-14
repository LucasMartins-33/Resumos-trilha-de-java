# Capítulo 03: Code Structure, Comments & Formatting (Estrutura, Comentários e Formatação)

## 1. A Verdade sobre Comentários
A regra de ouro sobre comentários no Clean Code é: **Na maior parte do tempo, você deve evitá-los.** 
Comentários frequentemente são usados para compensar um código ruim ou mal nomeado. Além disso, quando o código muda, os desenvolvedores muitas vezes esquecem de atualizar os comentários, gerando **desinformação**.

### ❌ Comentários Ruins (O que não fazer)
1. **Informação Redundante:** O código deve explicar a si mesmo com nomes claros.
   ```javascript
   // Ruim: Comentário desnecessário
   // Conecta no banco de dados se o driver for SQL
   if (driver === 'SQL') {
       dbEngine.connect();
   }
   ```
2. **Divisores de Blocos:** Usar comentários para desenhar "linhas" ou separar áreas de globals, imports e métodos polui o código visualmente. Se um arquivo precisa de muitas divisões, ele provavelmente está muito grande e deveria ser dividido em arquivos menores.
   ```python
   # --- GLOBALS ---
   x = 10
   # ---------------
   ```
3. **Comentários Enganosos (Misleading):** Um comentário que diz uma coisa, mas a função faz outra. O comentário passa a mentir para o desenvolvedor.
4. **Código Comentado:** Deixar blocos inteiros de código comentados "só para caso precise no futuro" é uma péssima prática. Atualmente usamos sistemas de controle de versão (como o **Git**). Se você não usa mais um código, **apague-o**. O Git sempre guardará o histórico.

### ✅ Comentários Bons (Exceções à regra)
1. **Avisos Legais/Licenças:** Blocos obrigatórios por empresa/projeto no topo do arquivo (ex: Licença MIT, Copyright).
2. **Explicações Críticas (Ex: RegEx):** Coisas intrinsecamente difíceis de ler, onde um nome de variável não é suficiente.
   ```javascript
   // Checa o padrão numérico específico do formulário da Receita Federal
   const taxRegex = /^[0-9]{3}\.[0-9]{3}\.[0-9]{3}\-[0-9]{2}$/; 
   ```
3. **Avisos Estruturais/Técnicos (Warnings):** Avisar colegas sobre uma limitação técnica ou side-effect.
   ```javascript
   // AVISO: Só funciona no navegador. Não roda no backend (Node.js)
   const data = localStorage.getItem('user');
   ```
4. **Notas de TODO:** Aceitáveis temporariamente durante o desenvolvimento para marcar pendências, mas não deixe que se acumulem eternamente no projeto.
5. **Documentação Pública (Docstrings/JSDoc):** Essencial se você está criando uma API ou biblioteca que outras pessoas irão consumir. O seu editor (IDE) utilizará esses comentários para dar auto-complete para os usuários.

---

## 2. Formatação de Código (Code Formatting)
Formatar o código corretamente transporta significado, agrupa lógicas e transforma seu código numa "redação" com um fluxo suave, sem que a pessoa que lê precise "pular" pelo arquivo para tentar entender o fluxo.

### 📜 Formatação Vertical (De cima para baixo)
Organização do uso das linhas em seu arquivo.
1. **O Tamanho do Arquivo:** Arquivos muito grandes, que possuem muitos "conceitos" diferentes, devem ser divididos. **Regra de bolso:** Se possível, declare apenas uma Classe por arquivo.
2. **Espaçamento (Blank Lines):** A linha em branco é uma vírgula/ponto-parágrafo no código. Separe conceitos diferentes com linhas em branco (ex: os imports, o construtor, os métodos), mas mantenha lógicas estreitamente correlacionadas agrupadas.
   ```javascript
   // Sem espaçamento vira um bloco denso difícil de processar
   import fs from 'fs';
   class Storage {
       constructor() {}
       insert() {}
       delete() {}
   }
   ```
3. **Mantenha Métodos Relacionados Próximos:** Se o método A chama o método B, tente declará-los um após o outro, na ordem em que são lidos, para evitar a necessidade de "rolar a tela" freneticamente.

### ↔️ Formatação Horizontal (Esquerda para Direita)
Organização do que acontece dentro de uma mesma linha de código. O principal vilão aqui é a **barra de rolagem horizontal** (Horizontal Scrolling).
1. **Indentação:** Sempre respeite o guia de estilo da linguagem (seja 2 espaços, 4 espaços, ou tab). Código sem indentação é ininteligível.
2. **Quebre Linhas Longas (Break Long Statements):** Em vez de colocar chamadas encadeadas imensas em uma só linha, quebre em múltiplas linhas menores usando variáveis intermediárias.
   ```python
   # 🔴 Ruim (Linha comprida demais):
   # if user.get_active_subscription().get_plan().price > 50 and user.status == 'ACTIVE':
   
   # 🟢 Bom (Usando variáveis):
   current_plan = user.get_active_subscription().get_plan()
   is_premium = current_plan.price > 50
   
   if is_premium and user.status == 'ACTIVE':
       # ...
   ```
3. **Nomes Descritivos, porém Práticos:** Não deixe sua linha longa demais por causa de um nome absurdo como `storagePathForStoringImagesInATemporaryFolderForTheYear2020`. O nome precisa ser objetivo.

---

## 3. Particularidades das Linguagens (Language-specific Considerations)
- Algumas regras dependem da linguagem que você usa. Siga sempre o Guia de Estilos oficial (ex: *PEP8* no Python, *Airbnb Style Guide* no JS).
- **Hoisting (JavaScript vs Python):** No JavaScript, funções declaradas com `function` são elevadas ("hoisted") para o topo. Você pode chamar uma função antes da linha em que ela foi declarada. No **Python**, isso geraria um erro; a ordem importa e o Python exige que a função exista antes de ser invocada, o que dita diretamente como você fará sua Formatação Vertical.
