📘 Capítulo 15: Modelando Estruturas (Diagramas Estruturais)

**O Cenário:**
O arquiteto te pediu para documentar as conexões profundas do sistema legado. 

🟢 Atividade 15.1: Visibilidade de Classe
1. Descreva como desenhar a classe `Funcionario`.
2. Coloque `+ getNome(): String` e `- salarioBase: double`. Certifique-se de usar os símbolos certos.

🟢 Atividade 15.2: Multiplicidade em Associações
1. Você tem a classe `Empresa` e a classe `Funcionario`.
2. Desenhe uma associação bidirecional (linha reta).
3. Na ponta da Empresa, marque `1`. Na ponta do Funcionario, marque `0..*` (uma empresa pode ter zero ou muitos funcionários).

🟡 Atividade 15.3: Agregação em Java e UML
1. O Losango Vazado representa Agregação. Uma `Turma` agrega vários `Alunos`.
2. Descreva ou codifique como isso é implementado em Java (uma lista que é passada no construtor ou por método, e não instanciada fixamente dentro da Turma).

🟡 Atividade 15.4: Composição (O Ciclo de Vida Rígido)
1. O Losango Preenchido representa Composição. Um `Carro` compõe um `Motor`.
2. Se o carro é destruído (Garbage Collector), o Motor morre junto.
3. No código Java, demonstre instanciando `this.motor = new Motor();` DIRETAMENTE no construtor da classe `Carro`.

🟠 Atividade 15.5: Diagrama de Objetos (Instâncias Físicas)
1. Se a classe é `Carro`, o objeto é notado como `fusca: Carro` com o nome sublinhado!
2. Especifique valores de atributos concretos para o objeto (ex: `cor = azul`).

🟠 Atividade 15.6: Interfaces Providas e Requeridas (Componentes)
1. Componente A é um Microserviço. Componente B é um Gateway de Pagamento.
2. Como representar no Diagrama de Componentes que A "requer" a interface B? (Usando a notação do semi-circulo - *Socket* e do círculo completo - *Ball*).

🔴 Atividade 15.7: Dependências entre Pacotes
1. Crie os pacotes `com.loja.ui` e `com.loja.domain`.
2. Marque a seta tracejada do `ui` para o `domain` significando que a UI depende do domínio, demonstrando arquitetura limpa em UML.

🔴 Atividade 15.8: Nós Físicos no Deployment
1. Crie um `Node` chamado "Servidor AWS EC2" representado por uma caixa 3D (cubo).
2. Dentro dele, coloque o Artefato (Arquivo físico real) `app-loja.jar`.

🔴 Atividade 15.9: Estrutura Composta (Composite Structure)
1. Se um subsistema é uma "Caixa Preta", use o Diagrama de Estrutura Composta.
2. Descreva um componente com uma "Porta" (um quadradinho na borda da caixa). Tudo que entra/sai do componente deve obrigatoriamente cruzar esta porta delegando internamente.

🔴 Atividade 15.10: Unificando o Blueprint (Projeto)
1. Escreva um sumário defendendo por que usar Diagramas Estruturais combinados gera o "Blueprint" (planta-baixa definitiva) antes de construir o prédio (ou sistema monolítico complexo).
