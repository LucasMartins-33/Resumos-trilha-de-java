# Capítulo 10: AOP - Aspect-Oriented Programming (Programação Orientada a Aspectos)

Este resumo aborda de forma completa o capítulo de **AOP (Aspect-Oriented Programming)** com **Spring Boot 4**:
* Problema de Negócio: **Code Tangling** (emaranhamento de código) e **Code Scattering** (dispersão de código).
* Solução com AOP e o padrão **Proxy (Design Pattern)**.
* Conceitos Fundamentais: **Aspect**, **Advice**, **Pointcut**, **JoinPoint** e **Weaving**.
* Comparativo entre **Spring AOP** vs **AspectJ**.
* Tipos de Advice: `@Before`, `@AfterReturning`, `@AfterThrowing`, `@After` (finally), e `@Around`.
* Sintaxe detalhada de **Pointcut Expressions** e uso de Wildcards (`*` e `..`).
* Declarações Reutilizáveis de Pointcuts (`@Pointcut`) e Combinação com Operadores Lógicos (`&&`, `||`, `!`).
* Controle de Ordenação de Aspectos com `@Order`.
* Interceptação de Metadados e Argumentos com `JoinPoint` e `ProceedingJoinPoint`.
* Tratamento, alteração e re-lançamento de Exceções com AOP.

---

## 📌 1. O Problema de Negócio e a Solução AOP

Em aplicações enterprise, tarefas transversais à regra de negócio principal (como logging, segurança, auditoria e transações) acabam sendo duplicadas em múltiplos métodos e camadas.

### O Problema:
1. **Code Tangling (Emaranhamento)**: A regra de negócio principal (ex: `addAccount()`) fica poluída com códigos de infraestrutura (segurança, logs, medição de tempo).
2. **Code Scattering (Dispersão)**: Para alterar a forma de logar ou validar segurança, é necessário modificar dezenas ou centenas de arquivos/classes em todo o projeto.

### A Solução AOP:
AOP permite isolar esses **Cross-Cutting Concerns** (Interesses Transversais) em módulos reutilizáveis chamados **Aspectos**, que são injetados declarativamente sem modificar o código-fonte das classes de negócio.

```
+-------------------------------------------------------------------+
|                         MAIN APPLICATION                          |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
|                        SPRING AOP PROXY                           |
|  (Executa os Aspectos: LoggingAspect, SecurityAspect, etc.)       |
+-------------------------------------------------------------------+
                                  |
                                  v
+-------------------------------------------------------------------+
|                          TARGET OBJECT                            |
|  (Regra de Negócio Pura: AccountDAO.addAccount())                  |
+-------------------------------------------------------------------+
```

---

## 📌 2. Conceitos e Terminologia de AOP

| Termo | Descrição |
| :--- | :--- |
| **Aspect** | Módulo/Classe contendo código de infraestrutura transversal (ex: `MyDemoLoggingAspect`). Marcado com `@Aspect` e `@Component`. |
| **Advice** | A ação executada pelo Aspecto e **QUANDO** ela deve ser aplicada (`@Before`, `@AfterReturning`, `@AfterThrowing`, `@After`, `@Around`). |
| **Pointcut** | Expressão/Predicado que define **ONDE** o Advice deve interceptar a execução (ex: `execution(* com.luv2code.aopdemo.dao.*.*(..))`). |
| **JoinPoint** | Ponto específico durante a execução do programa (como a chamada de um método) onde o aspecto é aplicado. Fornece metadados (`getSignature()`, `getArgs()`). |
| **Weaving** | Processo de vincular aspectos aos objetos alvos (*Target Objects*). No Spring AOP, é feito em **tempo de execução (Runtime Weaving)** via Proxies do Spring. |

---

## 📌 3. Comparativo: Spring AOP vs AspectJ

| Característica | Spring AOP | AspectJ |
| :--- | :--- | :--- |
| **Abordagem** | Leve, focada em resolver os problemas mais comuns do Spring. | Framework AOP completo e robusto (padrão de mercado desde 2001). |
| **Weaving** | Runtime Weaving via **Spring Dynamic Proxies**. | Compile-time, Post-compile, ou Load-time Weaving. |
| **Join Points Suportados** | Suporta **apenas execução de métodos** em Beans gerenciados pelo Spring. | Suporta métodos, construtores, acesso a atributos (fields), etc. |
| **Complexidade** | Simples de configurar (incluso no `spring-boot-starter-aop`). | Exige compilador especial (AspectJ compiler - `ajc`) e sintaxe mais complexa. |
| **Performance** | Altíssima performance, com custo insignificante em runtime. | Performance marginalmente superior em execuções ultra pesadas. |

### Dependência Maven no Spring Boot:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

---

## 📌 4. Sintaxe de Pointcut Expressions (`execution`)

A estrutura completa de um ponto de corte `execution` é:
```
execution(modifiers-pattern? return-type-pattern declaring-type-pattern? method-name-pattern(param-pattern) throws-pattern?)
```
*(Itens com `?` são opcionais)*

### Exemplos Práticos de Pointcuts:

1. **Qualquer método `addAccount()` específico da classe `AccountDAO`**:
   `execution(public void com.luv2code.aopdemo.dao.AccountDAO.addAccount())`
2. **Qualquer método iniciado com `add` em qualquer classe**:
   `execution(public void add*())`
3. **Qualquer retorno e qualquer método que inicie com `add`**:
   `execution(* add*())`
4. **Método que recebe um parâmetro do tipo específico `Account`**:
   `execution(* add*(com.luv2code.aopdemo.Account))`
5. **Método com primeiro parâmetro `Account` e zero ou mais parâmetros adicionais**:
   `execution(* add*(com.luv2code.aopdemo.Account, ..))`
6. **Qualquer número de parâmetros (zero ou vários)**:
   `execution(* add*(..))`
7. **TODOS os métodos de QUALQUER classe no pacote `com.luv2code.aopdemo.dao`**:
   `execution(* com.luv2code.aopdemo.dao.*.*(..))`

---

## 📌 5. Declaração Reutilizável de Pointcuts (`@Pointcut`) e Operadores Lógicos

Podemos declarar expressões de Pointcut em métodos vazios para reutilizá-los e combiná-los usando os operadores `&&` (AND), `||` (OR) e `!` (NOT).

### Criando uma Classe Central de Pointcuts Reutilizáveis (`LuvAopExpressions.java`):

```java
package com.luv2code.aopdemo.aspect;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class LuvAopExpressions {

    // Reutilizável: Todos os métodos do pacote DAO
    @Pointcut("execution(* com.luv2code.aopdemo.dao.*.*(..))")
    public void forDaoPackage() {}

    // Reutilizável: Todos os Getters do pacote DAO
    @Pointcut("execution(* com.luv2code.aopdemo.dao.*.get*(..))")
    public void getter() {}

    // Reutilizável: Todos os Setters do pacote DAO
    @Pointcut("execution(* com.luv2code.aopdemo.dao.*.set*(..))")
    public void setter() {}

    // COMBINAÇÃO: Todos os métodos do pacote DAO, EXCETO Getters e Setters
    @Pointcut("forDaoPackage() && !(getter() || setter())")
    public void forDaoPackageNoGetterSetter() {}
}
```

---

## 📌 6. Ordenação de Aspectos (`@Order`)

Quando múltiplos aspectos interceptam o mesmo método, a ordem de execução pode ser definida separando os aspectos em classes distintas e utilizando a anotação **`@Order(N)`**.

* **Menores números possuem maior prioridade** (ex: `@Order(1)` executa antes de `@Order(2)`).
* Números negativos são permitidos (ex: `@Order(-10)`).
* A ordem para o retorno do método é executada na sequência inversa (pilha).

```java
@Aspect
@Component
@Order(1) // Executa em 1º lugar no pré-processamento
public class MyCloudLogAsyncAspect {
    @Before("com.luv2code.aopdemo.aspect.LuvAopExpressions.forDaoPackageNoGetterSetter()")
    public void logToCloud() {
        System.out.println("\n===> Logging to Cloud in async mode");
    }
}

@Aspect
@Component
@Order(2) // Executa em 2º lugar
public class MyDemoLoggingAspect {
    @Before("com.luv2code.aopdemo.aspect.LuvAopExpressions.forDaoPackageNoGetterSetter()")
    public void beforeAddAccountAdvice() {
        System.out.println("\n===> Executing @Before advice on method");
    }
}
```

---

## 📌 7. Tipos de Advices & Exemplos de Código

### 7.1 Advice `@Before` + Interceptação de Argumentos (`JoinPoint`)

Executa **ANTES** da chamada do método.

```java
@Aspect
@Component
@Order(3)
public class MyApiAnalyticsAspect {

    @Before("com.luv2code.aopdemo.aspect.LuvAopExpressions.forDaoPackageNoGetterSetter()")
    public void performApiAnalytics(JoinPoint theJoinPoint) {
        System.out.println("\n===> Performing API analytics");

        // 1. Acessando a assinatura do método
        MethodSignature methodSig = (MethodSignature) theJoinPoint.getSignature();
        System.out.println("Method Signature: " + methodSig);

        // 2. Acessando os argumentos passados na chamada do método
        Object[] args = theJoinPoint.getArgs();
        for (Object tempArg : args) {
            System.out.println("Arg value: " + tempArg);
            if (tempArg instanceof Account) {
                Account theAccount = (Account) tempArg;
                System.out.println("Account Name: " + theAccount.getName());
            }
        }
    }
}
```

---

### 7.2 Advice `@AfterReturning` (Pós-processamento e Modificação de Dados)

Executa **APÓS a conclusão bem-sucedida** do método (sem exceções). Permite inspecionar ou até **modificar/enriquecer** o valor de retorno.

```java
@Aspect
@Component
public class MyDemoLoggingAspect {

    @AfterReturning(
        pointcut = "execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))",
        returning = "result" // Nome da variável que receberá o retorno do método
    )
    public void afterReturningFindAccountsAdvice(JoinPoint theJoinPoint, List<Account> result) {
        System.out.println("\n===> Executing @AfterReturning advice on: " + theJoinPoint.getSignature());
        System.out.println("Result original: " + result);

        // Modificando os dados antes de chegarem ao chamador
        if (result != null && !result.isEmpty()) {
            for (Account tempAccount : result) {
                tempAccount.setName(tempAccount.getName().toUpperCase());
            }
        }
        System.out.println("Result modificado (Uppercase): " + result);
    }
}
```

---

### 7.3 Advice `@AfterThrowing` (Auditoria de Exceções)

Executa quando o método **lança uma Exceção**. Nota: O `@AfterThrowing` **NÃO intercepta/abafa** a exceção, ela continuará propagando para o chamador.

```java
@Aspect
@Component
public class MyDemoLoggingAspect {

    @AfterThrowing(
        pointcut = "execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))",
        throwing = "theExc" // Nome do parâmetro contendo a Exceção lançada
    )
    public void afterThrowingFindAccountsAdvice(JoinPoint theJoinPoint, Throwable theExc) {
        System.out.println("\n===> Executing @AfterThrowing on method: " + theJoinPoint.getSignature());
        System.out.println("Exception capturada pelo Aspecto: " + theExc.getMessage());
    }
}
```

---

### 7.4 Advice `@After` (Finally)

Executa **SEMPRE** que o método finaliza (seja por sucesso ou por exceção). Funciona idêntico ao bloco `finally` do Java. Não tem acesso direto ao retorno ou à exceção.

```java
@Aspect
@Component
public class MyDemoLoggingAspect {

    @After("execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))")
    public void afterFinallyFindAccountsAdvice(JoinPoint theJoinPoint) {
        System.out.println("\n===> Executing @After (finally) on method: " + theJoinPoint.getSignature());
    }
}
```

---

### 7.5 Advice `@Around` (Controle Total de Execução, Timers e Tratamento de Exceções)

O advice mais poderoso. Executa **ANTES E DEPOIS** da chamada do método. Utiliza `ProceedingJoinPoint` para controlar se o método real deve ser invocado (`proceed()`).

#### Usos Comuns:
1. Medição de tempo de execução (Profiling/Benchmarking).
2. Abafar/Tratar Exceções (impedindo a propagação para a aplicação principal).
3. Re-lançar exceções customizadas.

#### Exemplo 1: Medindo Tempo de Execução com `@Around`
```java
@Aspect
@Component
public class MyDemoLoggingAspect {

    @Around("execution(* com.luv2code.aopdemo.service.TrafficFortuneService.getFortune(..))")
    public Object aroundGetFortune(ProceedingJoinPoint theProceedingJoinPoint) throws Throwable {
        System.out.println("\n===> Executing @Around on method: " + theProceedingJoinPoint.getSignature());

        long begin = System.currentTimeMillis();

        // Executa o método original
        Object result = null;
        try {
            result = theProceedingJoinPoint.proceed();
        } catch (Exception e) {
            System.out.println("@Around capturou a exceção: " + e.getMessage());
            // Opção A: Re-lançar a exceção -> throw e;
            // Opção B: Abafar a exceção e retornar uma mensagem amigável de fallback:
            result = "Major traffic! Heavy highway backup, but response recovered by AOP!";
        }

        long end = System.currentTimeMillis();
        long duration = end - begin;

        System.out.println("====> Tempo total de execução: " + duration + " ms");

        return result;
    }
}
```

---

## 📋 Tabela Resumo das Anotações AOP

| Anotação | Local de Uso | Propósito / Descrição |
| :--- | :--- | :--- |
| **`@Aspect`** | Classe | Marca a classe Java como um Aspecto AOP do Spring. |
| **`@Before("pointcut")`** | Método | Executa o código **antes** da execução do método-alvo. |
| **`@AfterReturning`** | Método | Executa **após o método retornar com sucesso**. Dá acesso ao objeto retornado (`returning="..."`). |
| **`@AfterThrowing`** | Método | Executa **após o método lançar uma exceção**. Dá acesso à exceção (`throwing="..."`). |
| **`@After`** | Método | Executa **sempre** ao final do método (análogo ao bloco `finally`). |
| **`@Around`** | Método | Executa **antes e depois** do método. Recebe `ProceedingJoinPoint` e controla a chamada `proceed()`. |
| **`@Pointcut("expressão")`** | Método | Declara uma expressão de pointcut nomeada para ser reutilizada e combinada (`&&`, `||`, `!`). |
| **`@Order(N)`** | Classe Aspecto | Define a precedência de execução entre múltiplos aspectos (menor número = maior prioridade). |
| **`JoinPoint`** | Parâmetro de Advice | Dá acesso aos metadados do método interceptado (`getSignature()`, `getArgs()`). |
| **`ProceedingJoinPoint`** | Parâmetro em `@Around` | Subclasse de `JoinPoint` usada no `@Around` para invocar o método alvo via `.proceed()`. |
