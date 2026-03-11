## Application Context

Let’s go deep into Spring Boot internals and see how `ApplicationContext` **is created step-by-step** when a Spring Boot application starts.

---

### 1️⃣ Starting Point of Every Spring Boot App

Every Spring Boot app starts with:

```@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

}
 ```

The most important line is:

`SpringApplication.run()`

This method is responsible for **bootstrapping the entire Spring container**.

---

### 2️⃣ What Happens Inside SpringApplication.run()

Internally Spring Boot executes roughly this flow:

```
run()
 ├── prepareEnvironment()
 ├── createApplicationContext()
 ├── prepareContext()
 ├── refreshContext()
 └── runApplicationRunners()
```

---

### 3️⃣ Step 1 — Prepare Environment

Spring loads configuration from:

```
application.properties
application.yml
environment variables
command-line arguments
```

Example:

```
server.port=8080
spring.datasource.url=...
```

Internally Spring builds an object:

`ConfigurableEnvironment`

Example code internally:

`ConfigurableEnvironment environment = new StandardEnvironment();`

This environment stores all **configuration values**.

---

### 4️⃣ Step 2 — Create ApplicationContext

Spring Boot now decides **what type of ApplicationContext to create**.

Depending on the application type:

| Application Type | Context                               |
| ---------------- | ------------------------------------- |
| Web Application  | `ServletWebServerApplicationContext`  |
| Reactive App     | `ReactiveWebServerApplicationContext` |
| Non-Web App      | `AnnotationConfigApplicationContext`  |

Example creation:

`ApplicationContext context = new AnnotationConfigApplicationContext();`

or for web apps:

`ServletWebServerApplicationContext`

This context is still **empty at this stage**.

---

### 5️⃣ Step 3 — Load Bean Definitions
Spring scans classes using:

```@Component
@Service
@Repository
@Controller
@Configuration
```

Example:

```
@Service
public class UserService {}
```

Spring creates a **BeanDefinition** object.

BeanDefinition contains metadata:

```
BeanDefinition
 ├── beanClass
 ├── scope
 ├── lazyInit
 ├── constructor args
 └── dependencies
```

Example internally:

`UserService.class → BeanDefinition`

These definitions are stored inside:

`DefaultListableBeanFactory`

Important concept:

`Spring does NOT create objects yet
It only registers metadata`

---

### 6️⃣ Step 4 — Register Infrastructure Beans

Spring registers many internal beans automatically.

Examples:

```
AutowiredAnnotationBeanPostProcessor
ConfigurationClassPostProcessor
CommonAnnotationBeanPostProcessor
EventListenerMethodProcessor
```

These power features like:

```
@Autowired
@PostConstruct
@EventListener
@Configuration
```

---

7️⃣ Step 5 — Process @Configuration Classes
Spring processes classes like:

```
@Configuration
public class AppConfig {

    @Bean
    public UserService userService() {
        return new UserService();
    }

}
```

Spring parses these using:

`ConfigurationClassPostProcessor`

It converts `@Bean` methods into **BeanDefinitions**.

---

### 8️⃣ Step 6 — Component Scanning

Because of:

`@SpringBootApplication`

Spring performs:

`@ComponentScan`

It scans packages for:

```
@Component
@Service
@Repository
@Controller
```

Example:

`com.parvez.app`

Spring registers each discovered bean as **BeanDefinition**.

---

### 9️⃣ Step 7 — BeanFactory Post Processing
Before creating beans, Spring allows modification of bean definitions.
Example interface:

`BeanFactoryPostProcessor`

Used by:

```
@Configuration
Spring Boot auto configuration
Property placeholder resolution
```

Example internal processor:

`PropertySourcesPlaceholderConfigurer`

This resolves:

`${database.url}`

---

### 🔟 Step 8 — Register BeanPostProcessors

These intercept bean lifecycle events.

Important Interface

`BeanPostProcessor`

Example methods:

```
postProcessBeforeInitialization()

postProcessAfterInitialization()
```

Used by features like:

```
AOP
@Transactional
@Autowired
```
---

### 1️⃣1️⃣ Step 9 — Create Singleton Beans

Now Spring actually creates beans.
This happens inside:

`finishBeanFactoryInitialization()`

Which calles:

`preInstantiateSingletons()`

Flow:

`BeanDefinition → Object instance`

Example:

`UserService.class → new UserService()`

Dependencies are injected.

Example:

```
@Service
class OrderService {

    @Autowired
    UserService userService;

}
```

Spring resolves the dependency automatically.

---

### 1️⃣2️⃣ Dependency Injection Process

Steps:

```
1. Create object
2. Inject dependencies
3. Run BeanPostProcessor
4. Call init methods
5. Bean ready
```

Example Lifecycle:

```
Constructor
↓
@Autowired injection
↓
@PostConstruct
↓
BeanPostProcessor
↓
Ready
```

---

### 1️⃣3️⃣ Start Embedded Web Server

If this is a **web application**, Spring Boot now starts

```
Tomcat
Jetty
Undertow
```

Example:

`Tomcat started on port 8080`

Internally:

`ServletWebServerFactory`

creates the server.

---

### 1️⃣4️⃣ Publish Application Events

Spring publishes lifecycle events:

```
ApplicationStartingEvent
ApplicationEnvironmentPreparedEvent
ApplicationPreparedEvent
ContextRefreshedEvent
ApplicationStartedEvent
ApplicationReadyEvent
```

These allow hooks for frameworks and developers.

Example listener:

```
@EventListener
public void handleReady(ApplicationReadyEvent event) {
    System.out.println("Application started");
}
```

---

### 1️⃣5️⃣ Run ApplicationRunner / CommandLineRunner

Spring Boot finally runs:

```
CommandLineRunner
ApplicationRunner
```

Example:

```
@Component
public class StartupRunner implements CommandLineRunner {

    public void run(String... args) {
        System.out.println("Application started");
    }

}
```

---

### Complete Startup Flow

```
main()
 ↓
SpringApplication.run()
 ↓
Prepare Environment
 ↓
Create ApplicationContext
 ↓
Load Bean Definitions
 ↓
Register BeanPostProcessors
 ↓
Process @Configuration
 ↓
Component Scan
 ↓
BeanFactoryPostProcessor
 ↓
Create Singleton Beans
 ↓
Start Web Server
 ↓
Publish Events
 ↓
Run ApplicationRunner
 ↓
Application Ready
```

---

### Real Internal Core Classes

Important Spring classes:

```
SpringApplication
AbstractApplicationContext
DefaultListableBeanFactory
ConfigurationClassPostProcessor
AutowiredAnnotationBeanPostProcessor
AbstractAutowireCapableBeanFactory
```

These are the **heart of Spring Framework**.

---
### Important Insigh

Spring container consists of **two core parts**:

```
ApplicationContext
       ↓
BeanFactory
```

Think:

```
ApplicationContext → Operating System
BeanFactory → CPU
Beans → Applications
```

---

### Final Takeaway

When you run a Spring Boot app:

`SpringApplication.run()`

Spring Internally:

```
Creates ApplicationContext
Loads BeanDefinitions
Creates Beans
Injects Dependencies
Starts Web Server
Runs Application
```