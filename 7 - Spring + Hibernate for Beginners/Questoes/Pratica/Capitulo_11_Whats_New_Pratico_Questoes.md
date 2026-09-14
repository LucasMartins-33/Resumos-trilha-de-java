# Questões Práticas – Capítulo 11: What’s New in Spring Boot 4

## Exercício 11.1 – Configurar **Kubernetes Liveness Probe** no `application.yaml`
**Cenário**
Adicione a configuração padrão de **livenessProbe** usando o endpoint `/actuator/health/liveness`.

```yaml
management:
  endpoint:
    health:
      probes:
        enabled: true # 🟢 Atividade: habilitar probes
  endpoints:
    web:
      exposure:
        include: health,info # expor health para k8s
```

---

## Exercício 11.2 – Criar **Dockerfile** multistage para Spring Boot 4 native image
**Cenário**
Construa a aplicação como **native image** usando GraalVM e crie uma imagem Docker mínima.

```dockerfile
# ---------- Stage 1: Build native binary ----------
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY . .
# Instalação do GraalVM native-image (via sdkman ou direct download)
RUN curl -L https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-21.0.2/graalvm-ce-java21-linux-amd64-21.0.2.tar.gz \
    | tar xz && mv graalvm-ce-java21-* /opt/graalvm
ENV PATH="/opt/graalvm/bin:${PATH}"
RUN gu install native-image
# Build native executable
RUN ./mvnw -Pnative native:compile

# ---------- Stage 2: Runtime ----------
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/target/*.jar /app/app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app/app.jar"]
```

---

## Exercício 11.3 – Habilitar **Modular classpath** (feature de Spring Boot 4)
**Cenário**
Altere o `pom.xml` para usar `spring-boot-modules` e remover dependências desnecessárias.

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>4.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- Exemplo de dependência modular -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId> <!-- 🟢 Atividade: usar módulo WebMVC -->
</dependency>
```

---

## Exercício 11.4 – Utilizar **Null‑Safety** com `@NonNull` nas propriedades de bean
**Cenário**
Aplique a anotação `@NonNull` (do pacote `org.springframework.lang`) em um bean de serviço e configure a compilação para falhar caso haja `null`.

```java
@Service
public class GreetingService {
    private final MessageSource messages;

    public GreetingService(@NonNull MessageSource messages) { // 🟢 Atividade: contrato non‑null
        this.messages = messages;
    }

    public String greet(String name) {
        Objects.requireNonNull(name, "name must not be null"); // 🟢 Atividade: verificação explícita
        return messages.getMessage("greeting", new Object[]{name}, Locale.getDefault());
    }
}
```

---

## Exercício 11.5 – Configurar **Observability** com Micrometer + OpenTelemetry
**Cenário**
Adicione a dependência `micrometer-tracing-bridge-otel` e registre um span customizado em um método de serviço.

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
    <version>1.13.0</version>
</dependency>
```

```java
@Service
public class OrderService {
    private final Tracer tracer; // 🟢 Atividade: injetar tracer OpenTelemetry
    public OrderService(Tracer tracer) { this.tracer = tracer; }

    public void process(Order order) {
        Span span = tracer.nextSpan().name("processOrder").start();
        try (Tracer.SpanInScope ws = tracer.withSpan(span)) {
            // lógica do pedido
        } finally {
            span.end(); // 🟢 Atividade: encerrar span
        }
    }
}
```

---

## Exercício 11.6 – Expor **Custom Actuator Endpoint** `info/build`
**Cenário**
Crie um endpoint que devolve informações de versão e data de build.

```java
@Component
@Endpoint(id = "build") // 🟢 Atividade: endpoint custom
public class BuildEndpoint {
    @ReadOperation
    public Map<String, String> buildInfo() {
        return Map.of(
            "version", "4.0.0",
            "timestamp", Instant.now().toString()
        );
    }
}
```

---

## Exercício 11.7 – Configurar **Graceful Shutdown** (tempo de 30 s)
**Cenário**
Ajuste `application.yaml` para que o Spring Boot aguarde 30 s antes de encerrar.

```yaml
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s # 🟢 Atividade: tempo de shutdown
```

---

## Exercício 11.8 – Migrar de **Jackson 2** para **Jackson 3**
**Cenário**
Substitua a dependência `com.fasterxml.jackson.core:jackson-databind` por `com.fasterxml.jackson.core:jackson-databind:3.x` e ajuste a classe de configuração.

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>3.0.0</version> <!-- 🟢 Atividade: versão 3 -->
</dependency>
```

```java
@Configuration
public class JacksonConfig {
    @Bean
    public JsonMapper jsonMapper() { // 🟢 Atividade: usar JsonMapper da v3
        return JsonMapper.builder()
            .findAndAddModules()
            .build();
    }
}
```

---

## Exercício 11.9 – Configurar **OpenAPI 3.1** com Springdoc
**Cenário**
Atualize a dependência para a versão que suporta OpenAPI 3.1 e personalize o título da API.

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.5.0</version> <!-- já suporta 3.1 -->
</dependency>
```

```java
@OpenAPIDefinition(
    info = @Info(title = "Demo API", version = "4.0", description = "What’s new in Spring Boot 4"))
@Configuration
public class OpenApiConfig {}
```

---

## Exercício 11.10 – Testar **Kubernetes Deployment** com probes via `kubectl`
**Cenário**
Crie um manifesto `deployment.yaml` que usa as probes configuradas e execute `kubectl apply`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: demo-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: demo
  template:
    metadata:
      labels:
        app: demo
    spec:
      containers:
        - name: demo
          image: demo:latest
          ports:
            - containerPort: 8080
          livenessProbe:
            httpGet:
              path: /actuator/health/liveness
              port: 8080
            initialDelaySeconds: 15
            periodSeconds: 20
          readinessProbe:
            httpGet:
              path: /actuator/health/readiness
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
```

```bash
# Aplicar no cluster
kubectl apply -f deployment.yaml
# Verificar probes
kubectl get pods && kubectl describe pod <pod-name>
```

---

**Como usar**
1. Crie o pacote `com.example.newboot` e adicione as classes acima.
2. Preencha os blocos marcados com 🟢/🟡 conforme necessário.
3. Compile com `mvn spring-boot:build-image` (para native) ou `mvn package` e teste as novas funcionalidades.

> **Arquivo salvo em** `C:\Users\lucas\OneDrive\Documentos\Cursos\spring-boot-4-spring-7-hibernate-for-beginners\Questoes\Pratica\Capitulo_11_Whats_New_Pratico_Questoes.md`
