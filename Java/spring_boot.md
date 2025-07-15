
# Spring Boot

[[spring_boot_guide|Spring Boot Guide]]



![[Pasted image 20250711234942.png]]


![[Pasted image 20250711235128.png]]


![[Pasted image 20250711235222.png]]


![[Pasted image 20250711235341.png]]



![[Pasted image 20250711235522.png]]







**Spring Boot Starters** - обычные зависимости (jars), которые добавляют нужный функционал в проект и следуют определенному шаблону наименования: spring-boot-starter-*


![[Pasted image 20250711235618.png]]


![[Pasted image 20250711235705.png]]



Все настройки сохраняются в одном файле `application.properties`

Шаблоны (например Thymeleaf) лежат в папке `templates`



## Dependency Management


Одна из проблем в разработке приложений - это несовместимость версий различных зависимостей.

**Для ее решения каждый Spring Boot release разрабатывается и тестируется с определенным набором third-party зависимостей**


![[Pasted image 20250503231945.png]]




## @Conditional


### 🔹 Что такое `@Conditional` в Spring Boot?

Аннотация `@Conditional` говорит Spring:  
**"Создавай этот бин только при определённом условии."**

---

### 📦 Пример из жизни:

Представь, ты пишешь приложение, которое может работать **с локальной БД** или **облачной БД**, в зависимости от настроек.

Ты не хочешь всегда создавать оба подключения. Хочешь, чтобы Spring **создавал только то, что нужно**.

Вот тут и помогает `@Conditional`.

---

### 🔧 Пример кода
```java
@Configuration
public class DatabaseConfig {

    @Bean
    @Conditional(LocalDatabaseCondition.class)
    public DataSource localDataSource() {
        // подключение к локальной БД
    }

    @Bean
    @Conditional(CloudDatabaseCondition.class)
    public DataSource cloudDataSource() {
        // подключение к облачной БД
    }
}
```


### 📘 Что такое `LocalDatabaseCondition`?

Это просто класс, который говорит: **условие выполняется или нет**.
```java
public class LocalDatabaseCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        String type = context.getEnvironment().getProperty("db.type");
        return "local".equals(type);
    }
}
```

Если в `application.properties` будет:
```java
db.type=local
```

то Spring создаст `localDataSource()`. Если будет `cloud`, то — `cloudDataSource()`.

---

### ✅ Как это помогает?

- Можно включать/отключать бины в зависимости от настроек, окружения, профиля, операционной системы и т.д.
    
- Код становится **гибким** и **расширяемым**.
    

---

### 🧠 Запомни:

- `@Conditional` — это способ **включать/отключать бины** по условиям.
    
- В Spring Boot есть много готовых аннотаций на базе `@Conditional`, например:
    
    - `@ConditionalOnProperty`
        
    - `@ConditionalOnClass`
        
    - `@ConditionalOnMissingBean`





## Настройка проекта на Spring Boot


![[Pasted image 20250503235651.png]]
базовое Spring Boot приложение

```java
@SpringBootApplication  
public class DemoApplication {  
  
    public static void main(String[] args) {  
        SpringApplication.run(DemoApplication.class, args);  
    }  
  
}
```






## Lombok




Вместо создания конструктора для final полей можно использовать аннотацию `@RequiredArgsConstructor`:
```java
@Service  
@RequiredArgsConstructor  
public class UserService {  
    private final UserRepository userRepository;  
  
}
```








## YAML format



**YAML(yet another markup language)** - это аналог properties.

Вот пример:
```yaml
db:  
  username: maxim  
  pool:  
    size: 5
```

А вот как это бы выглядело в poperties:
```properties
db.username=maxim  
db.pool.size=5
```



## @ConfigurationProperties

![[Pasted image 20250507190042.png]]


Есть файл yaml:
```yaml
db:  
  username: maxim  
  password: 3maxim14  
  url: postgres:5432  
  hosts: localhost,127.0.0.1  
  pool:  
    size: 5  
    timeout: 10  
  pools:  
    - size: 1  
      timeout: 1  
    - size: 2  
      timeout: 2  
    - size: 3  
      timeout: 3  
  properties:  
    firts: 123  
    second: 456  
    third.value: 789
```


На основе его создаем класс DatabaseProperties:
```java
@Data  
@NoArgsConstructor  
public class DatabaseProperties {  
    private String username;  
    private String password;  
    private String url;  
    private String hosts;  
    private PoolProperties pool;  
    private List<PoolProperties> pools;  
    private Map<String, Object> properties;  
  
  
    @Data  
    @NoArgsConstructor    public static class PoolProperties {  
        private Integer size;  
        private Integer timeout;  
    }  
}
```



### Но как сделать так чтобы Spring понял связь между этим классом  и yaml-файлом?

1. Создать бин в каком-нибудь конфигурационном файле и пометить его аннотацией `@ConfigurationProperties`:
```java
@Bean
@ConfigurationProperties(prefix = "db")
public DatabaseProperties databaseProperties() {
	return new DatabaseProperties();
}
```


2. Указать над классом `@ConfigurationProperties(prefix = "db")` но тогда его надо будет пометить аннотацией `@Component`:
```java
@Data  
@NoArgsConstructor  
@Component  
@ConfigurationProperties(prefix="db")  
public class DatabaseProperties {  
    private String username;  
    private String password;  
    private String url;  
    private String hosts;  
    private PoolProperties pool;  
    private List<PoolProperties> pools;  
    private Map<String, Object> properties;  
  
  
    @Data  
    @NoArgsConstructor    public static class PoolProperties {  
        private Integer size;  
        private Integer timeout;  
    }  
}
```

Либо можно указать над классом где метод main аннотацию `@ConfigurationPropertiesScan`:
```java
@SpringBootApplication  
@ConfigurationPropertiesScan  
public class ApplicationRunner {  
    public static void main(String[] args) {  
        try(ConfigurableApplicationContext context = SpringApplication.run(ApplicationRunner.class, args)) {  
            System.out.println(context.getBean(DatabaseProperties.class));  
        }  
    }  
}
```
И над самим классом указать `@ConfigurationProperties` но уже без `@Component`:
```java
@Data  
@NoArgsConstructor  
@ConfigurationProperties(prefix="db")  
public class DatabaseProperties {  
    private String username;  
    private String password;  
    private String url;  
    private String hosts;  
    private PoolProperties pool;  
    private List<PoolProperties> pools;  
    private Map<String, Object> properties;  
  
  
    @Data  
    @NoArgsConstructor    public static class PoolProperties {  
        private Integer size;  
        private Integer timeout;  
    }  
}
```



Также можно сделать при помощи record:
![[Pasted image 20250507191434.png]]






# Logging Starter


![[Pasted image 20250507193605.png]]


![[Pasted image 20250507193634.png]]


Нужно над классом в котором выводим логи указать аннотацию `@Slf4j`








# Консольный запуск Spring Boot приложения


`./mvnw package` - собрать проект если нет maven на сервере

И далее перейти в папку `target` и ввести команду:
```shell
java -jar name-of-your-jar-file.jar
```





# Более сложное Spring Boot приложение


Сначала устанавливаем конфигурацию в `application.properties`:

```shell
spring.application.name=spring-crud-app-boot  
  
# Конфигурация источника данных (Data source)  
spring.datasource.driver-class-name=org.postgresql.Driver  
spring.datasource.url=jdbc:postgresql://localhost:5432/practice  
spring.datasource.username=maxim  
spring.datasource.password=3maxim14  
  
  
# Конфигурация Hibernate  
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.PostgreSQLDialect  
spring.jpa.properties.hibernate.show_sql=true  
  
  
# Чтобы работали HTTP методы PUT, PATCH, DELETE  
spring.mvc.hiddenmethod.filter.enabled=true
```



