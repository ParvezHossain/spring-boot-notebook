### prepareEnvironment()

`prepareEnvironment()` is one of the **most important early steps** in Spring Boot startup. It happens **before the** `ApplicationContext` **is refreshed and before beans are created**.

When Spring Boot starts, it needs to collect **all configuration values** for the application.

Examples:

```
server.port=8080
spring.datasource.url=jdbc:mysql://localhost/test
logging.level.root=INFO
```

These values can come from many places:

- application.properties
- application.yml
- OS Environment variables
- Command-line arguments
- .env
- Default values

Spring gathers them into a single object called:

`Environment`

Specifically:

`ConfigurableEnvironment`

Example:

```
Environment env = context.getEnvironment();

String port = env.getProperty("server.port");
```

So **prepareEnvironment() builds this environment object**.

---

### The Actual Method Flow

Inside `SpringApplication.run()`:

```
SpringApplication.run()
        ↓
prepareEnvironment()
```

Simplified internal code:

```
private ConfigurableEnvironment prepareEnvironment(
        SpringApplicationRunListeners listeners,
        ApplicationArguments applicationArguments) {

    ConfigurableEnvironment environment = getOrCreateEnvironment();

    configureEnvironment(environment, applicationArguments.getSourceArgs());

    listeners.environmentPrepared(environment);

    bindToSpringApplication(environment);

    return environment;
}
```

---

Let’s break each step.

### Step 1 — Create Environment Object

Spring first creates an Environment implementation.

Depending on the application type:

| App Type    | Environment                      |
| ----------- | -------------------------------- |
| Non-web     | `StandardEnvironment`            |
| Servlet web | `StandardServletEnvironment`     |
| Reactive    | `StandardReactiveWebEnvironment` |


Example:

`ConfigurableEnvironment environment = new StandardServletEnvironment();`

This object will store all configuration sources.

---

### Step 2 — Property Sources

Spring Environment contains **PropertySources**.

Think of it like a **priority list of configuration providers**.

Structure:

```
Environment
   └── PropertySources
           ├── systemProperties
           ├── systemEnvironment
           ├── application.properties
           ├── commandLineArgs
           └── defaultProperties
```

Example internally:

`MutablePropertySources`

Each entry is a `PropertySource`.

Example:

```
PropertySource
   name = systemEnvironment
   source = OS variables
```
---

### Step 3 — Add System Properties

Spring automatically loads:

**JVM properties**

`-Dserver.port=8080`

Stored in:

`systemProperties`

Example Access:

`System.getProperty("server.port");`

---

### Step 4 — Load OS Environment Variables

Example:

```
DB_HOST=localhost
PORT=8080
```

Stored as:

`systemEnvironment`

Spring maps them like:

`DB_HOST → db.host`

This allows:

`${db.host}`

---

### Command Line Arguments

Example:

`java -jar app.jar --server.port=9090`

Spring parses int into:

`commandLineArgs`

Highest priority source.

Example access: 

`env.getProperty("server.port")`

---

### Load application.properties / YAML

Spring boot loads configuration files from:

```
classpath:/application.properties
classpath:/application.yml
file:./config/
file:./
```

Priority order example:

```
application-dev.yml
application.yml
```

Spring loads them using:

`ConfigDataEnvironmentPostProcessor`

Internally:

`OriginTrackedMapPropertySource`

---

### Profiles Processing

Spring now resolves active profiles.

Example:

`spring.profiles.active=dev`

This activies:

`application-dev.yml`

Profiles allow different configuration for environments:

```
dev
test
prod
```

Example YAML:

```
spring:
  profiles: dev
server:
  port: 8081
```

---

### Run EnvironmentPostProcessors

Spring Boot allows modification of environment before context creation.

Interface:

`EnvironmentPostProcessor`

Used internally by:

```
Spring Boot ConfigData loader
Cloud Config
Vault
Docker integrations
```

Example:

`ConfigDataEnvironmentPostProcessor`

This loads:

```
application.yml
application.properties
profile configs
```

---

### Notify SpringApplicationRunListeners

Spring now fires event:

`ApplicationEnvironmentPreparedEvent`

This allows frameworks to hook into environment creation.

Listeners can modify properties.

Example:

`LoggingApplicationListener`

It configures logging based on:

```
logging.level
logging.file
```

---

### Bind Environment to SpringApplication


Spring binds configuration values to:

`SpringApplication`

Example properties:

```
spring.main.banner-mode
spring.main.lazy-initialization
spring.main.web-application-type
```


Binding example:

`spring.main.lazy-initialization=true`

Spring sets:

`SpringApplication.lazyInitialization = true`

This uses:

`Binder API`

---

### Property Resolution Priority (Very Important)

Order of precedence:

```
1. Command line arguments
2. Java System properties
3. OS Environment variables
4. application-{profile}.properties
5. application.properties
6. default properties
```

Example:

`server.port=8080`

If you run:

`java -jar app.jar --server.port=9000`

Final value:

`9000`