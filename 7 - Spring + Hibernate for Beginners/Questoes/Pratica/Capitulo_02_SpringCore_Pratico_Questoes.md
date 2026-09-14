# Questões Práticas – Capítulo 02: Spring Core

## Exercício 2.1 – Configurar Inversão de Controle (IoC) com `@Configuration`
**Cenário**
Crie uma classe de configuração que exponha dois beans simples (`String` e `Integer`).

```java
package com.example.core;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration // 🟢 Atividade: marcar classe como fonte de beans
public class CoreConfig {

    @Bean // 🟢 Atividade: definir bean String
    public String helloMessage() {
        return "Olá Spring Core";
    }

    @Bean // 🟢 Atividade: definir bean Integer
    public Integer answer() {
        return 42;
    }
}
```

---

## Exercício 2.2 – Demonstrar **Constructor Injection**
**Cenário**
Crie um serviço `GreetingService` que receba o bean `String` via construtor.

```java
package com.example.core.service;

import org.springframework.stereotype.Service;

@Service // 🟢 Atividade: marcar como bean Spring
public class GreetingService {
    private final String message; // 🟢 Atividade: campo final

    public GreetingService(String message) { // 🟢 Atividade: construtor com injeção
        this.message = message;
    }

    public String greet() {
        return message + " from GreetingService";
    }
}
```

---

## Exercício 2.3 – Demonstrar **Setter Injection**
**Cenário**
Crie um componente `CounterComponent` que recebe um `Integer` via setter.

```java
package com.example.core.component;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component // 🟢 Atividade: marcar como bean
public class CounterComponent {
    private Integer count;

    @Autowired // 🟢 Atividade: injetar via setter
    public void setCount(Integer count) {
        this.count = count;
    }

    public int current() {
        return count != null ? count : 0;
    }
}
```

---

## Exercício 2.4 – Demonstrar **Field Injection** (não recomendado, mas ilustrativo)
**Cenário**
Injete diretamente o bean `String` em um campo.

```java
package com.example.core.component;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class FieldInjectComponent {
    @Autowired // 🟢 Atividade: injetar campo diretamente
    private String helloMessage;

    public String getMessage() {
        return helloMessage;
    }
}
```

---

## Exercício 2.5 – Resolver **`NoUniqueBeanDefinitionException`** usando `@Qualifier`
**Cenário**
Crie duas implementações da interface `Formatter` (`UpperFormatter` e `LowerFormatter`). Use `@Qualifier` para escolher a desejada.

```java
public interface Formatter { String format(String input); }

@Component @Qualifier("upper") class UpperFormatter implements Formatter { public String format(String s){return s.toUpperCase();} }
@Component @Qualifier("lower") class LowerFormatter implements Formatter { public String format(String s){return s.toLowerCase();} }

@Service class FormatService {
    private final Formatter formatter;
    public FormatService(@Qualifier("upper") Formatter formatter) { // 🟢 Atividade: escolher bean
        this.formatter = formatter;
    }
    public String apply(String s){ return formatter.format(s); }
}
```

---

## Exercício 2.6 – Definir bean **`@Primary`**
**Cenário**
Defina `UpperFormatter` como bean primário e injete `Formatter` sem `@Qualifier`.

```java
@Component @Primary class UpperFormatter implements Formatter { … }
@Component class LowerFormatter implements Formatter { … }

@Service class PrimaryService {
    private final Formatter formatter;
    public PrimaryService(Formatter formatter) { // 🟢 Atividade: bean primário será usado
        this.formatter = formatter;
    }
}
```

---

## Exercício 2.7 – Explorar escopos **`singleton`** vs **`prototype`**
**Cenário**
Crie dois beans `SingletonBean` e `PrototypeBean` e verifique quantas instâncias são criadas ao solicitar o bean duas vezes.

```java
@Component @Scope("singleton") class SingletonBean { }
@Component @Scope("prototype") class PrototypeBean { }

@RestController
class ScopeTestController {
    @Autowired private SingletonBean s1;
    @Autowired private SingletonBean s2;
    @Autowired private PrototypeBean p1;
    @Autowired private PrototypeBean p2;

    @GetMapping("/scope")
    public Map<String, Boolean> test() {
        return Map.of(
            "sameSingleton", s1 == s2,
            "samePrototype", p1 == p2
        );
    }
}
```

---

## Exercício 2.8 – Utilizar **`@PostConstruct`** e **`@PreDestroy`**
**Cenário**
Crie um bean que escreva no console ao ser inicializado e antes de ser destruído.

```java
@Component
public class LifecycleBean {
    @PostConstruct // 🟢 Atividade: método chamado após construção
    public void init() { System.out.println("Bean inicializado"); }

    @PreDestroy // 🟢 Atividade: método chamado antes da destruição (singleton)
    public void cleanup() { System.out.println("Bean sendo destruído"); }
}
```

---

## Exercício 2.9 – Configuração Java usando **`@ConfigurationProperties`**
**Cenário**
Mapeie propriedades `app.name` e `app.version` para um POJO.

```properties
# application.properties
app.name=SpringCoreDemo
app.version=1.0
```

```java
@ConfigurationProperties(prefix = "app")
@Component // 🟢 Atividade: registrar como bean
public class AppInfo {
    private String name;
    private String version;
    // getters/setters
}

@RestController
class InfoController {
    @Autowired private AppInfo info;
    @GetMapping("/info")
    public Map<String,String> getInfo() {
        return Map.of("name", info.getName(), "version", info.getVersion());
    }
}
```

---

## Exercício 2.10 – Criar um **`@Bean`** que dependa de outro bean
**Cenário**
Defina um bean `RandomService` que recebe um `java.util.Random` bean já configurado.

```java
@Configuration
public class RandomConfig {
    @Bean // 🟢 Atividade: bean Random
    public Random random() { return new Random(); }

    @Bean // 🟢 Atividade: bean que depende de Random
    public RandomService randomService(Random random) {
        return new RandomService(random);
    }
}

public class RandomService {
    private final Random random;
    public RandomService(Random random) { this.random = random; }
    public int nextInt(int bound) { return random.nextInt(bound); }
}
```

---

**Como usar**
1. Crie a estrutura de pacotes (`com.example.core...`).
2. Preencha cada *TODO* marcado com 🟢 ou 🟡.
3. Rode `mvn spring-boot:run` e teste os endpoints (`/scope`, `/info`).

> **Arquivo salvo em** `C:\Users\lucas\OneDrive\Documentos\Cursos\spring-boot-4-spring-7-hibernate-for-beginners\Questoes\Pratica\Capitulo_02_SpringCore_Pratico_Questoes.md`
