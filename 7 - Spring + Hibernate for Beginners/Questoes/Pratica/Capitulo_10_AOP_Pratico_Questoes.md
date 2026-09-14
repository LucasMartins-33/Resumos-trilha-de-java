# Questões Práticas – Capítulo 10: AOP (Aspect‑Oriented Programming)

## Exercício 10.1 – @Before Advice simples
**Cenário**
Você tem um serviço `AccountService` que grava contas no banco. Quer logar um texto antes de executar qualquer método `addAccount(..)` do DAO.

```java
package com.luv2code.aopdemo.aspect;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LoggingAspect {

    // 🟢 Atividade 10.1.1 – Complete o pointcut que captura todos os métodos addAccount()
    @Pointcut("execution(* com.luv2code.aopdemo.dao.*.addAccount(..))")
    public void forAddAccount() {}

    // 🟡 Atividade 10.1.2 – Crie o @Before advice que imprime a assinatura
    @Before("forAddAccount()")
    public void beforeAddAccountAdvice(JoinPoint jp) {
        System.out.println("\n===> Executando @Before em: " + jp.getSignature());
    }
}
```

---

## Exercício 10.2 – @AfterReturning e modificação do retorno
**Cenário**
Um repositório `AccountDAO` tem o método `findAccounts()` que devolve `List<Account>`. Quer‑se transformar todos os nomes das contas para MAIÚSCULAS após a consulta.

```java
package com.luv2code.aopdemo.aspect;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;
import java.util.List;

@Aspect
@Component
public class UppercaseAspect {

    // 🟢 Atividade 10.2.1 – Pointcut que captura findAccounts()
    @Pointcut("execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))")
    public void findAccountsPointcut() {}

    // 🟡 Atividade 10.2.2 – @AfterReturning que recebe o resultado
    @AfterReturning(pointcut = "findAccountsPointcut()", returning = "result")
    public void afterReturningFindAccounts(JoinPoint jp, List<Account> result) {
        System.out.println("\n===> Executando @AfterReturning em: " + jp.getSignature());
        // Transformar nomes para MAIÚSCULA
        for (Account acc : result) {
            acc.setName(acc.getName().toUpperCase());
        }
        System.out.println("Resultado modificado: " + result);
    }
}
```

---

## Exercício 10.3 – @AfterThrowing para auditoria de exceções
**Cenário**
Quando `AccountDAO.findAccounts()` lançar exceção, registre‑a no log e relance uma exceção customizada `AccountNotFoundException`.

```java
package com.luv2code.aopdemo.aspect;

import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class ExceptionAspect {

    // 🟢 Atividade 10.3.1 – Pointcut para o mesmo método findAccounts()
    @Pointcut("execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))")
    public void findAccountsWithException() {}

    // 🟡 Atividade 10.3.2 – @AfterThrowing que captura a exceção original
    @AfterThrowing(pointcut = "findAccountsWithException()", throwing = "theExc")
    public void afterThrowingFindAccounts(Throwable theExc) {
        System.out.println("\n===> @AfterThrowing capturou: " + theExc.getMessage());
        // Log adicional pode ser adicionado aqui
        throw new AccountNotFoundException("Conta não encontrada", theExc);
    }
}
```

---

## Exercício 10.4 – @After (finally) para log de término
**Cenário**
Independentemente de sucesso ou falha, o método `AccountDAO.findAccounts()` deve registrar a hora de término da execução.

```java
package com.luv2code.aopdemo.aspect;

import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;
import java.time.LocalDateTime;

@Aspect
@Component
public class FinallyAspect {

    // 🟢 Atividade 10.4.1 – Pointcut para findAccounts()
    @Pointcut("execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))")
    public void findAccountsPointcut() {}

    // 🟡 Atividade 10.4.2 – @After que imprime a hora de término
    @After("findAccountsPointcut()")
    public void afterFinallyFindAccounts() {
        System.out.println("\n===> Finalizado em: " + LocalDateTime.now());
    }
}
```

---

## Exercício 10.5 – @Around para medição de tempo
**Cenário**
Meça o tempo gasto pela chamada ao método `TrafficFortuneService.getFortune()` e exiba o total em milissegundos.

```java
package com.luv2code.aopdemo.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class TimingAspect {

    // 🟢 Atividade 10.5.1 – Pointcut que captura getFortune()
    @Pointcut("execution(* com.luv2code.aopdemo.service.TrafficFortuneService.getFortune(..))")
    public void fortunePointcut() {}

    // 🟡 Atividade 10.5.2 – @Around que mede o tempo
    @Around("fortunePointcut()")
    public Object aroundGetFortune(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        long end = System.currentTimeMillis();
        System.out.println("\n===> Tempo total de execução: " + (end - start) + " ms");
        return result;
    }
}
```

---

## Exercício 10.6 – Reuso de @Pointcut e operadores lógicos
**Cenário**
Crie um ponto central que captura **todos** os métodos do pacote `dao` exceto getters e setters. Use‑o em dois aspects diferentes.

```java
package com.luv2code.aopdemo.aspect;

import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class LuvAopExpressions {

    @Pointcut("execution(* com.luv2code.aopdemo.dao.*.*(..))")
    public void forDaoPackage() {}

    @Pointcut("execution(* com.luv2code.aopdemo.dao.*.get*(..))")
    public void getter() {}

    @Pointcut("execution(* com.luv2code.aopdemo.dao.*.set*(..))")
    public void setter() {}

    // Combinação sem getters e setters
    @Pointcut("forDaoPackage() && !(getter() || setter())")
    public void forDaoPackageNoGetterSetter() {}
}
```

---

## Exercício 10.7 – Controle de ordenação com @Order
**Cenário**
Dois aspects devem rodar antes do método DAO: primeiro `CloudLogAspect`, depois `DemoLoggingAspect`.

```java
package com.luv2code.aopdemo.aspect;

import org.aspectj.lang.annotation.*;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Aspect
@Component
@Order(1) // Executa primeiro
public class CloudLogAspect {
    @Before("com.luv2code.aopdemo.aspect.LuvAopExpressions.forDaoPackageNoGetterSetter()")
    public void logToCloud() {
        System.out.println("\n===> Logging to Cloud (async)");
    }
}

@Aspect
@Component
@Order(2) // Executa depois
public class DemoLoggingAspect {
    @Before("com.luv2code.aopdemo.aspect.LuvAopExpressions.forDaoPackageNoGetterSetter()")
    public void beforeAddAccountAdvice() {
        System.out.println("\n===> Executando @Before advice em método");
    }
}
```

---

## Exercício 10.8 – JoinPoint para inspeção de argumentos
**Cenário**
Antes da execução de qualquer método `add*()` no DAO, imprima o tipo e valor de todos os argumentos.

```java
package com.luv2code.aopdemo.aspect;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class ApiAnalyticsAspect {

    // 🟢 Pointcut para métodos que começam com "add"
    @Pointcut("execution(* com.luv2code.aopdemo.dao.*.add*(..))")
    public void addMethods() {}

    // 🟡 @Before que itera sobre os argumentos
    @Before("addMethods()")
    public void performApiAnalytics(JoinPoint jp) {
        System.out.println("\n===> Executando API Analytics");
        Object[] args = jp.getArgs();
        for (Object arg : args) {
            System.out.println("Argumento: " + arg + " (tipo: " + arg.getClass().getSimpleName() + ")");
        }
    }
}
```

---

## Exercício 10.9 – @Around com tratamento de exceção e fallback
**Cenário**
`TrafficFortuneService.getFortune()` pode lançar `RuntimeException`. Crie um @Around que, caso a exceção ocorra, retorne a string de fallback.

```java
package com.luv2code.aopdemo.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class FortuneFallbackAspect {

    // 🟢 Pointcut para getFortune()
    @Pointcut("execution(* com.luv2code.aopdemo.service.TrafficFortuneService.getFortune(..))")
    public void fortuneServicePointcut() {}

    // 🟡 @Around que captura exceção e devolve fallback
    @Around("fortuneServicePointcut()")
    public Object aroundGetFortune(ProceedingJoinPoint pjp) throws Throwable {
        try {
            return pjp.proceed();
        } catch (Exception e) {
            System.out.println("@Around capturou exceção: " + e.getMessage());
            return "Default Fortune – Service Unavailable";
        }
    }
}
```

---

## Exercício 10.10 – Aspecto usando annotation customizada @TrackTime
**Cenário**
Crie a annotation `@TrackTime` e um aspect que mede o tempo de execução de qualquer método anotado.

```java
// 1️⃣ Annotation – TrackTime.java
package com.luv2code.aopdemo.annotation;

import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface TrackTime {}
```

```java
// 2️⃣ Aspect – TrackTimeAspect.java
package com.luv2code.aopdemo.aspect;

import com.luv2code.aopdemo.annotation.TrackTime;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.*;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class TrackTimeAspect {

    // 🟢 Pointcut que captura any method annotated with @TrackTime
    @Pointcut("@annotation(com.luv2code.aopdemo.annotation.TrackTime)")
    public void trackTimeAnnotation() {}

    // 🟡 @Around que mede e imprime a duração
    @Around("trackTimeAnnotation()")
    public Object aroundTrackTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        long end = System.currentTimeMillis();
        System.out.println(pjp.getSignature() + " executado em " + (end - start) + " ms");
        return result;
    }
}
```

### Como testar
1. Anote qualquer método de serviço, por exemplo `AccountService.addAccount(..)`, com `@TrackTime`.
2. Rode a aplicação e verifique a saída no console.

---

### 📦 Onde salvar
**Caminho:** `C:\Users\lucas\OneDrive\Documentos\Cursos\spring-boot-4-spring-7-hibernate-for-beginners\Questoes\Pratica\Capitulo_10_AOP_Pratico_Questoes.md`

---

> **Observação:** Cada exercício contém as atividades marcadas com círculos coloridos (🟢, 🟡, 🟠) para facilitar o acompanhamento.
