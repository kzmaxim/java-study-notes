
![[Pasted image 20250606201019.png]]




# MVC (Model-View-Controller)


![[Pasted image 20250606201052.png]]


![[Pasted image 20250606201316.png]]




![[Pasted image 20250606201446.png]]




![[Pasted image 20250606201555.png]]



![[Pasted image 20250606201626.png]]



![[Pasted image 20250606201651.png]]



# Первое приложение на Spring MVC

## Tomcat


Для того чтобы работал сайт нам нужен какой-то сервер. `Tomcat` это как раз такой сервер. 

Чтобы связать IntelliJ IDEA с Tomcat нужно сначала скачать и распаковать архив.
После этого выбрать RUN -> Edit Configurations и добавить Tomcat. 


Если Tomcat не запускается через IDE то нужно изменить права доступа у файла Tomcat:
![[Pasted image 20250606203423.png]]





## Зависимости

Чтобы работать с Spring MVC нужно добавить зависимости: `spring-core`, `spring-context`, `spring-web`, `spring-webmvc`

Также нужно добавить шаблонизатор Thymeleaf-spring5:
```xml
<dependency>  
  <groupId>org.thymeleaf</groupId>  
  <artifactId>thymeleaf-spring6</artifactId>  
  <version>3.1.2.RELEASE</version>  
</dependency>
```






## Web.xml


Чтобы сервер обращался к `Dispatcher servlet` нужно сконфигурировать web.xml файл:
```xml
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         id="WebApp_ID" version="3.1">

    <display-name>spring-mvc-app</display-name>

    <absolute-ordering/>

    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/applicationContextMVC.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

</web-app>
```


И также нужно настроить файл `applicationContextMVC.xml`:
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xmlns:context="http://www.springframework.org/schema/context"  
       xmlns:mvc="http://www.springframework.org/schema/mvc"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans  
       http://www.springframework.org/schema/beans/spring-beans.xsd       http://www.springframework.org/schema/context       http://www.springframework.org/schema/context/spring-context.xsd       http://www.springframework.org/schema/mvc       http://www.springframework.org/schema/mvc/spring-mvc.xsd">  
  
    <context:component-scan base-package="com.maxim.springcourse"/>  
  
    <mvc:annotation-driven/>  
  
    <bean id="templateResolver" class="org.thymeleaf.spring6.templateresolver.SpringResourceTemplateResolver">  
        <property name="prefix" value="/WEB-INF/views/"/>  
        <property name="suffix" value=".html"/>  
    </bean>  
  
    <bean id="templateEngine" class="org.thymeleaf.spring6.SpringTemplateEngine">  
        <property name="templateResolver" ref="templateResolver"/>  
        <property name="enableSpringELCompiler" value="true"/>  
    </bean>  
  
</beans>
```



## Controller


Вот пример контроллера:
```java
@Controller  
public class HelloController {  
  
    @GetMapping("/hello-world")  
    public String sayHello() {  
        return "hello_world";  
    }  
}
```

То есть при обращении по url `/hello-world` контроллер перенаправит на представление `hello_world.html`


Для создания контроллера используется аннотация `@Controller`
А аннотация `@GetMapping` нужна для GET-запросов 



## View

View хранится в WEB-INF/views

Это просто html-файл:
```html
<!doctype html>  
<html lang="ru">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport"  
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">  
    <meta http-equiv="X-UA-Compatible" content="ie=edge">  
    <title>Spring MVC App</title>  
</head>  
<body>  
<h1>Hello World!</h1>  
</body>  
</html>
```


Результат:

![[Pasted image 20250606210322.png]]






# Конфигурация с помощью Java кода


![[Pasted image 20250606232630.png]]


![[Pasted image 20250606232724.png]]



Для начала создаем конфигурационный файл который заменит файл `applicationContextMVC.xml`:
```java
@Configuration  
@ComponentScan("com.maxim.springcourse")  
@EnableWebMvc  
public class SpringConfig implements WebMvcConfigurer {  
    private final ApplicationContext applicationContext;  
  
    @Autowired  
    public SpringConfig(ApplicationContext applicationContext) {  
        this.applicationContext = applicationContext;  
    }  
  
    @Bean  
    public SpringResourceTemplateResolver templateResolver() {  
        SpringResourceTemplateResolver templateResolver = new SpringResourceTemplateResolver();  
        templateResolver.setApplicationContext(applicationContext);  
        templateResolver.setPrefix("/WEB-INF/views/");  
        templateResolver.setSuffix(".html");  
        return templateResolver;  
    }  
  
    @Bean  
    public SpringTemplateEngine templateEngine() {  
        SpringTemplateEngine templateEngine = new SpringTemplateEngine();  
        templateEngine.setTemplateResolver(templateResolver());  
        templateEngine.setEnableSpringELCompiler(true);  
        return templateEngine;  
    }  
  
    @Override  
    public void configureViewResolvers(ViewResolverRegistry registry) {  
        ThymeleafViewResolver resolver = new ThymeleafViewResolver();  
        resolver.setTemplateEngine(templateEngine());  
        registry.viewResolver(resolver);  
    }  
}
```



И теперь настраиваем класс который будет заменять файл `web.xml`:
```java
public class MySpringMVCDispatcherServletInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {  
    @Override  
    protected Class<?>[] getRootConfigClasses() {  
        return null;  
    }  
  
    @Override  
    protected Class<?>[] getServletConfigClasses() {  
        return new Class[] {SpringConfig.class};  
    }  
  
    @Override  
    protected String[] getServletMappings() {  
        return new String[] {"/"};  
    }  
}
```

И также нужно добавить зависимость `jakarta.servlet-api`:
```xml
<dependency>  
  <groupId>jakarta.servlet</groupId>  
  <artifactId>jakarta.servlet-api</artifactId>  
  <version>6.1.0</version>  
</dependency>
```








# Контроллеры. Аннотация @Controller



![[Pasted image 20250607000927.png]]


![[Pasted image 20250607001102.png]]



![[Pasted image 20250607001341.png]]




Пример контроллера:
```java
@Controller  
public class FirstController {  
  
    @GetMapping("/hello")  
    public String helloPage() {  
        return "first/hello";  
    }  
  
    @GetMapping("/goodbye")  
    public String goodByePage() {  
        return "first/goodbye";  
    }  
}
```








# Протокол HTTP. Методы GET и POST


## Клиент-серверная архитектура

![[Pasted image 20250607153054.png]]



## Запрос (request)

![[Pasted image 20250607153140.png]]



![[Pasted image 20250607153853.png]]

### GET

![[Pasted image 20250607153933.png]]



![[Pasted image 20250607154050.png]]



### POST


![[Pasted image 20250607154215.png]]



![[Pasted image 20250607154300.png]]




![[Pasted image 20250607154418.png]]




## Ответ (response)


![[Pasted image 20250607154613.png]]


![[Pasted image 20250607154841.png]]


![[Pasted image 20250607155311.png]]










# Параметры GET запроса. Аннотация @RequestParam


![[Pasted image 20250608203331.png]]



Пример метода который использует параметры:
```java
@GetMapping("/hello")  
public String helloPage(HttpServletRequest request) {  
    String name = request.getParameter("name");  
    String surname = request.getParameter("surname");  
  
    System.out.println("Hello, " + name + " " + surname);  
  
    return "first/hello";  
}
```


Либо так:

```java
@GetMapping("/hello")  
public String helloPage(@RequestParam("name") String name,  
                        @RequestParam("surname") String surname) {  
  
    System.out.println("Hello, " + name + " " + surname);  
  
    return "first/hello";  
}
```


#### **Интересный факт:**

Если при первом варианте не указать параметры, то spring просто подставит в переменные значения null. Если же также не указать параметры строки во втором варианте, то будет ошибка 400. Чтобы избежать этой ошибки нужно указать `required=false`:

```java
@GetMapping("/hello")  
public String helloPage(@RequestParam(value = "name", required = false) String name,  
                        @RequestParam(value = "surname", required = false) String surname) {  
  
    System.out.println("Hello, " + name + " " + surname);  
  
    return "first/hello";  
}
```









# Модель. Передача данных от контроллера к представлению


![[Pasted image 20250610194336.png]]



![[Pasted image 20250610194426.png]]




#### Пример: 

Создаем контроллер и передаем в методе модель (Model):
```java
@GetMapping("/hello")  
public String helloPage(@RequestParam(value = "name", required = false) String name,  
                        @RequestParam(value = "surname", required = false) String surname,  
                        Model model // модель) {  
  
    model.addAttribute("message", "Hello, " + name + " " + surname);  
  
    return "first/hello";  
}
```

И уже в представлении с помощью Thymeleaf отображаем эти данные:
```html
<p th:text="${message}"/>
```



#### Еще пример: калькулятор
```java
@GetMapping("/calculator")  
public String calculator(@RequestParam(value="a") Integer a,  
                         @RequestParam(value="b") Integer b,  
                         @RequestParam(value="action") String action,  
                         Model model) {  
    Integer res;  
    if (action.equals("multiplication")) {  
        res = a * b;  
    } else if (action.equals("division")) {  
        res = a / b;  
    } else if (action.equals("addition")) {  
        res = a + b;  
    } else if (action.equals("subtraction")) {  
        res = a - b;  
    } else {  
        res = 0;  
    }  
  
    model.addAttribute("a", a);  
    model.addAttribute("b", b);  
    model.addAttribute("action", action);  
    model.addAttribute("result", res);  
  
    return "first/calculator";  
}
```


И отображаем все:
```html
<!DOCTYPE html>  
<html lang="ru" xmlns:th="http://www.thymeleaf.org">  
<head>  
    <meta charset="UTF-8">  
    <title>Calculator</title>  
</head>  
<body>  
<h1>Calculator</h1>  
<h4><span th:text="${a}"/> <span th:text="${action}"/> <span th:text="b"/> = <span th:text="${result}"/> </h4>  
</body>  
</html>
```








# CRUD. REST. Паттерн DAO (Data Access Object)



![[Pasted image 20250611210002.png]]



![[Pasted image 20250611210037.png]]



![[Pasted image 20250611210051.png]]



![[Pasted image 20250611210207.png]]



![[Pasted image 20250611210551.png]]


![[Pasted image 20250611210658.png]]




![[Pasted image 20250611213815.png]]





## Модель

```java
@Data  
@AllArgsConstructor  
public class Person {  
    private Integer id;  
    private String name;  
}
```


## DAO

```java
@Component
public class PersonDAO {  
    private static Integer peopleCount = 0;  
    private List<Person> people;  
  
  
    {  
        people = new ArrayList<>();  
  
        people.add(new Person(++peopleCount, "Alesha"));  
        people.add(new Person(++peopleCount, "Sasha"));  
        people.add(new Person(++peopleCount, "Alex"));  
        people.add(new Person(++peopleCount, "Tom"));  
  
    }  
  
  
    public List<Person> index() {  
        return people;  
    }  
  
    public Person show(Integer id) {  
        return people.stream().filter(p -> p.getId().equals(id)).findFirst().orElse(null);  
    }  
}
```



## Контроллер

```java
@Controller  
@RequestMapping("/people")  
public class PeopleController {  
  
    private PersonDAO personDAO;  
  
    @Autowired  
    public PeopleController(PersonDAO personDAO) {  
        this.personDAO = personDAO;  
    }  
  
    @GetMapping()  
    public String index(Model model) {  
        model.addAttribute("people", personDAO.index());  
        return "people/index";  
    }  
  
    @GetMapping("/{id}")  
    public String show(@PathVariable("id") Integer id, Model model) {  
        model.addAttribute("person", personDAO.show(id));  
        return "people/show";  
    }  
}
```








# Аннотация @ModelAttribute. HTML формы (Thymeleaf)



![[Pasted image 20250624173424.png]]



![[Pasted image 20250624173443.png]]



![[Pasted image 20250624173600.png]]




![[Pasted image 20250624173638.png]]



![[Pasted image 20250624173810.png]]



![[Pasted image 20250624173851.png]]


То ест аннотация `@ModelAttribute` над атрибутом занимается тем, что создаем объект необходимого класса и устанавливает туда необходимые значения




![[Pasted image 20250624174121.png]]





### Пример

Создаем сначала метод который по пути "people/new" будет возвращать нужное представление:
```java
@GetMapping("/new")  
public String newPerson(Model model) {  
    model.addAttribute("person", new Person());  
    return "people/new";  
}
```


Вот само это представление:
```html
<!DOCTYPE html>  
<html lang="ru" xmlns:th="http://www.thymeleaf.org">  
<head>  
    <meta charset="UTF-8">  
    <title>New Person</title>  
</head>  
<body>  
  
  
<form th:action="@{/people}" th:object="${person}" th:method="POST">  
    <label for="name">Name</label>  
    <input type="text" th:field="*{name}" id="name">  
  
    <input type="submit" value="Create!">  
</form>  
  
  
</body>  
</html>
```


Далее создаем метод `create` который будет создавать пользователя:

```java
@PostMapping()  
public String create(@ModelAttribute("person") Person person) {  
    personDAO.save(person);  
    System.out.println("Person created");  
  
    return "redirect:/people";  
}
```



И затем в personDAO создаем необходимый метод:
```java
public void save(Person person) {  
    person.setId(++peopleCount);  
    people.add(person);  
}
```









# CRUD приложение. Методы UPDATE, DELETE



![[Pasted image 20250626123805.png]]



![[Pasted image 20250626123820.png]]


![[Pasted image 20250626124028.png]]



## UPDATE

Сначала создадим метод контроллера который будет показывать отображение для редактирования:
```java
@GetMapping("/{id}/edit")  
public String edit(Model model, @PathVariable("id") Integer id) {  
    model.addAttribute("person", personDAO.show(id));  
    return "people/edit";  
}
```


Далее создадим это отображение:
```html
<!DOCTYPE html>  
<html lang="ru" xmlns:th="http://www.thymeleaf.org">  
<head>  
    <meta charset="UTF-8">  
    <title>Edit person</title>  
</head>  
<body>  
<form th:action="@{/people/{id}(id=${person.getId()})}" th:object="${person}" th:method="PATCH">  
    <label for="name">Name</label>  
    <input type="text" th:field="*{name}" id="name">  
  
    <input type="submit" value="Update!">  
</form>  
</body>  
</html>
```


Далее нужно подключить фильтр:
```java
@Override  
public void onStartup(ServletContext aServletContext) throws ServletException {  
    super.onStartup(aServletContext);  
    registerHiddenFieldFilter(aServletContext);  
}  
  
private void registerHiddenFieldFilter(ServletContext aContext) {  
    aContext.addFilter("hiddenHttpMethodFilter",  
            new HiddenHttpMethodFilter()).addMappingForUrlPatterns(  
            EnumSet.of(DispatcherType.REQUEST), true, "/*");  
}
```


И создать метод контроллера который принимает PATCH-запросы;
```java
@PatchMapping("/{id}")  
public String update(@ModelAttribute("person") Person person,  
                     @PathVariable("id") Integer id) {  
    personDAO.update(id, person);  
    return "redirect:/people";  
}
```


Ну и потом создать метод в DAO который обновляет человека:
```java
public void update(Integer id, Person person) {  
    Person personToBeUpdated = show(id);  
  
    personToBeUpdated.setName(person.getName());  
}
```


## DELETE

Также создаем метод контроллера который принимает DELETE-запросы:
```java
@DeleteMapping("/{id}")  
public String delete(@PathVariable("id") Integer id) {  
    personDAO.delete(id);  
    return "redirect:/people";  
}
```


Реализуем метод в DAO:
```java
public void delete(Integer id) {  
    people.removeIf(p -> p.getId().equals(id));  
}
```


И добавляем кнопку на страницу отображения пользователя:
```html
<!DOCTYPE html>  
<html lang="ru" xmlns:th="http://www.thymeleaf.org">  
<head>  
    <meta charset="UTF-8">  
    <title>Человек</title>  
</head>  
<body>  
<p th:text="${person.getName()}"/>  
<p th:text="${person.getId()}"/>  
<a th:href="@{/people/{id}/edit(id=${person.getId()})}">Edit</a>  
  
<form th:method="DELETE" th:action="@{/people/{id}(id=${person.getId()})}">  
    <input type="submit" value="Delete"/>  
</form>  
</body>  
</html>
```


И там самым получилось простое CRUD приложение!!!









# Валидация форм. Аннотация `@Valid`





Для валидации форм нужно подключить плагин `hibernate-validator`:
```xml
<dependency>  
  <groupId>org.hibernate.validator</groupId>  
  <artifactId>hibernate-validator</artifactId>  
  <version>8.0.1.Final</version>  
</dependency>
```


Эта зависимость позволяет использовать аннотации для валидации форм:
```java
@Data  
@Getter  
@AllArgsConstructor  
@NoArgsConstructor  
public class Person {  
    private Integer id;  
  
    @NotEmpty(message="Name should not be empty")  // не должно быть пустым
    @Size(min=2, max=30, message = "Name should be between 2 and 30") // размер 
    private String name;  
  
    @Min(value = 0, message="Age should be greater than zero")  // мин. значение
    private Integer age;  
  
    @NotEmpty(message="Email should not be empty")  
    @Email(message="Incorrect email!")  // email
    private String email;  
}
```




Также нужно поменять методы контроллера:
```java
@PostMapping()  
public String create(@ModelAttribute("person") @Valid Person person,  
                     BindingResult bindingResult) {  
    if (bindingResult.hasErrors()) {  
        log.warn("Binding error in post method: {}", bindingResult.getAllErrors().toString());  
        return "people/new";  
    }  
  
  
    personDAO.save(person);  
    log.info("Person created");  
  
    return "redirect:/people";  
}
```

Здесь используется аннотация `@Valid` которая как раз проверяет валидно ли передали данные при создании объекта. После атрибута с этой аннотацией обязательно нужно указать объект класса `BindingResult`

```java
@PatchMapping("/{id}")  
public String update(@ModelAttribute("person") @Valid Person person,  
                     BindingResult bindingResult,  
                     @PathVariable("id") Integer id) {  
    if (bindingResult.hasErrors()) {  
        log.warn("Binding error in patch method: {}", bindingResult.getAllErrors().toString());  
        return "people/edit";  
    }  
    personDAO.update(id, person);  
    return "redirect:/people";  
}
```


И нужно поменять саму форму чтобы она отображала ошибку если введены неправильно данные:
```html
<form th:action="@{/people}" th:object="${person}" th:method="POST">  
    <label for="name">Name</label>  
    <input type="text" th:field="*{name}" id="name">  
    <div style="color:red" th:if="${#fields.hasErrors('name')}" th:errors="*{name}">Name error</div>  
  
    <label for="age">Age</label>  
    <input type="text" th:field="*{age}" id="age">  
    <div style="color:red" th:if="${#fields.hasErrors('age')}" th:errors="*{age}">Name error</div>  
  
    <label for="email">Email</label>  
    <input type="text" th:field="*{email}" id="email">  
    <div style="color:red" th:if="${#fields.hasErrors('email')}" th:errors="*{email}">Name error</div>  
  
    <input type="submit" value="Create!">  
</form>
```







# JDBC API. Базы данных



![[Pasted image 20250627114636.png]]



![[Pasted image 20250627114712.png]]



Более подробно JDBC разобрана в этом конспекте [[JDBC]]

А пока вот как я реализовал отображение пользователей:
```java
public List<Person> index() {  
    ArrayList<Person> people = new ArrayList<>();  
  
    try {  
        Statement statement = connection.createStatement();  
        String SQL = "SELECT * FROM Person";  
        ResultSet resultSet = statement.executeQuery(SQL);  
  
        while(resultSet.next()) {  
            Person person = new Person();  
  
            person.setId(resultSet.getInt("id"));  
            person.setName(resultSet.getString("name"));  
            person.setAge(resultSet.getInt("age"));  
            person.setEmail(resultSet.getString("email"));  
  
            people.add(person);  
        }  
    } catch (SQLException e) {  
        e.printStackTrace();  
    }  
  
    return people;  
}
```


А до этого я подключился к базе данных вот так:
```java
private static final String URL = "jdbc:postgresql://localhost:5432/practice";  
private static final String USERNAME = "maxim";  
private static final String PASSWORD = "3maxim14";  
  
  
private static Connection connection;  
  
  
static {  
    try {  
        Class.forName("org.postgresql.Driver");  
        connection = DriverManager.getConnection(URL, USERNAME, PASSWORD);  
    } catch (ClassNotFoundException | SQLException e) {  
        e.printStackTrace();  
    }  
}
```



А вот так я сделал добавление нового пользователя:
```java
public void save(Person person) {  
    try {  
        Statement statement = connection.createStatement();  
        String SQL = String.format("INSERT INTO Person(id, name, age, email) VALUES (%d, '%s', %d, '%s')", 4, person.getName(), person.getAge(), person.getEmail());  
  
        statement.executeUpdate(SQL);  
    } catch (SQLException e) {  
        e.printStackTrace();  
    }  
}
```




# SQL injections и Prepared Statement


![[Pasted image 20250627125300.png]]



![[Pasted image 20250627125559.png]]



Пример использования `PreparedStatement` на примере метода `save`:
```java
public void save(Person person) {  
    try {  
        String SQL = "INSERT INTO Person (id, name, age, email) VALUES (5, ?, ?, ?)";  
        PreparedStatement statement = connection.prepareStatement(SQL);  
  
        statement.setString(1, person.getName());  
        statement.setInt(2, person.getAge());  
        statement.setString(3, person.getEmail());  
  
  
        statement.executeUpdate(SQL);  
    } catch (SQLException e) {  
        e.printStackTrace();  
    }  
}
```



Метод `show`:
```java
public Person show(Integer id) {  
    Person person = null;  
    try {  
        PreparedStatement preparedStatement = connection.prepareStatement("SELECT * FROM person WHERE id = ?");  
        preparedStatement.setInt(1, id);  
  
        ResultSet resultSet = preparedStatement.executeQuery();  
  
        resultSet.next();  
        person = new Person();  
        person.setId(resultSet.getInt("id"));  
        person.setName(resultSet.getString("name"));  
        person.setAge(resultSet.getInt("age"));  
        person.setEmail(resultSet.getString("email"));  
  
        return person;  
    } catch (SQLException e) {  
        e.printStackTrace();  
    }  
  
    return person;  
}
```

Метод `update`:
```java
public void update(Integer id, Person person) {  
    try {  
        PreparedStatement preparedStatement = connection.prepareStatement("UPDATE Person SET name=?, age=?, email=? WHERE id=?");  
        preparedStatement.setString(1, person.getName());  
        preparedStatement.setInt(2, person.getAge());  
        preparedStatement.setString(3, person.getEmail());  
        preparedStatement.setInt(4, id);  
  
        preparedStatement.executeUpdate();  
    } catch (SQLException e) {  
        e.printStackTrace();  
    }  
}
```

И метод `delete`:
```java
public void delete(Integer id) {  
    try {  
        PreparedStatement preparedStatement = connection.prepareStatement("DELETE FROM person WHERE id = ?");  
        preparedStatement.setInt(1, id);  
  
        preparedStatement.executeUpdate();  
    } catch (SQLException e) {  
        e.printStackTrace();  
    }  
}
```







# JDBC Template


![[Pasted image 20250627131501.png]]



![[Pasted image 20250627131548.png]]



![[Pasted image 20250627131623.png]]



![[Pasted image 20250627131702.png]]



Для начала нужно подключить зависимость `spring-jdbc`:
```xml
<dependency>  
  <groupId>org.springframework</groupId>  
  <artifactId>spring-jdbc</artifactId>  
  <version>6.2.7</version>  
</dependency>
```


Далее нужно создать бин `DataSource`. Это нужно для того чтобы Spring знал где база данных и как к ней подключиться:
```java
@Bean  
public DataSource dataSource() {  
    DriverManagerDataSource dataSource = new DriverManagerDataSource();  
  
    dataSource.setDriverClassName("org.postgresql.Driver");  
    dataSource.setUrl("jdbc:postgresql://localhost:5432/practice");  
    dataSource.setUsername("maxim");  
    dataSource.setPassword("3maxim14");  
  
    return dataSource;  
}
```


А затем нужно создать бин самого `JdbcTemplate`:
```java
@Bean  
public JdbcTemplate jdbcTemplate() {  
    return new JdbcTemplate(dataSource());  
}
```




После этого в DAO нужно внедрить объект `JdbcTemplate`:
```java
private final JdbcTemplate jdbcTemplate;  
  
@Autowired  
public PersonDAO(JdbcTemplate jdbcTemplate) {  
    this.jdbcTemplate = jdbcTemplate;  
}
```



## **Практика**


Перепишем метод `index`:
```java
public List<Person> index() {  
    return jdbcTemplate.query("SELECT * FROM Person", new PersonMapper());  
}
```

Чтобы работало нужно создать класс `PersonMapper`:
```java
public class PersonMapper implements RowMapper<Person> {  
  
    @Override  
    public Person mapRow(ResultSet resultSet, int rowNum) throws SQLException {  
        Person person = new Person();  
  
        person.setId(resultSet.getInt("id"));  
        person.setName(resultSet.getString("name"));  
        person.setAge(resultSet.getInt("age"));  
        person.setEmail(resultSet.getString("email"));  
  
        return person;  
    }  
}
```

Этот класс показывает как преобразовать строку в объект `Person`


Далее реализуем метод `show`:
```java
public Person show(Integer id) {  
    return jdbcTemplate.query("SELECT * FROM Person WHERE id = ?", new Object[]{id}, new PersonMapper())  
            .stream().findAny().orElse(null);  
}
```


Либо новый вариант:
```java
public Person show(Integer id) {  
    return jdbcTemplate.queryForObject("SELECT * FROM Person WHERE id = ?", new PersonMapper(), id);  
}
```


Также вместо `PersonMapper` можно использовать `BeanPropertyRowMapper`:
```java
new BeanPropertyRowMapper<>(Person.class)
```


И далее реализуем метод `save`:
```java
public void save(Person person) {  
    jdbcTemplate.update("INSERT INTO person VALUES (1, ?, ?, ?)", person.getName(), person.getAge(), person.getEmail());  
}
```

Метод `update`:
```java
public void update(Integer id, Person person) {  
    jdbcTemplate.update("UPDATE person SET name=? WHERE id=?", person.getName(), id);  
}
```

И метод `delete`:
```java
public void update(Integer id, Person person) {  
    jdbcTemplate.update("UPDATE person SET name=? WHERE id=?", person.getName(), id);  
}
```


И тем самым в CRUD приложение добавил работу с БД!!!!






# Конфигурация БД из внешнего файла




Для начала нужно создать файл `database.properties` в папке resources и задать значения для конфигурации БД:
```shell
driver=org.postgresql.Driver  
url=jdbc:postgresql://localhost:5432/practice  
username=maxim  
password=3maxim14
```

И также там же нужно создать файл `database.properties.origin` для того чтобы другие люди знали как заполнять файл `.properties`:
```shell
driver=  
url=  
username=  
password=
```



Далее, в классе помеченном как `@Configuration`  нужно указать путь к этому файлу: 
```java
@PropertySource("classpath:database.properties")
```


Также нужно внедрить переменную `environment` в этот класс:
```java
public class SpringConfig implements WebMvcConfigurer {  
    private final ApplicationContext applicationContext;  
    private final Environment environment;  // вот
  
    @Autowired  
    public SpringConfig(ApplicationContext applicationContext, Environment environment) {  
        this.applicationContext = applicationContext;  
        this.environment = environment;  
    }
```



И потом нужно поменять бин `DataSource` чтобы он брал данные из файла при помощи объекта `environment`:
```java
@Bean  
public DataSource dataSource() {  
    DriverManagerDataSource dataSource = new DriverManagerDataSource();  
  
    dataSource.setDriverClassName(Objects.requireNonNull(environment.getProperty("driver")));  
    dataSource.setUrl(environment.getProperty("url"));  
    dataSource.setUsername(environment.getProperty("username"));  
    dataSource.setPassword(environment.getProperty("password"));  
  
    return dataSource;  
}
```









# Пакетное обновление (Batch update)



![[Pasted image 20250630123014.png]]



![[Pasted image 20250630123652.png]]



Вот пример batch-вставки 1000 значений при помощи метода `batchUpdate()`:
```java
jdbcTemplate.batchUpdate("INSERT INTO person VALUES (?, ?, ?)", new BatchPreparedStatementSetter() {  
  
    @Override  
    public void setValues(PreparedStatement ps, int i) throws SQLException {  
        ps.setString(1, people.get(i).getName());  
        ps.setInt(2, people.get(i).getAge());  
        ps.setString(3, people.get(i).getEmail());  
    }  
  
    @Override  
    public int getBatchSize() {  
        return people.size();  
    }  
});
```












# Продвинутая валидация и Spring Validator




Для продвинутой валидации нужно сначала создать класс который имплементирует интерфейс `Validator`:
```java
@Component  
public class PersonValidator implements Validator {  
    private final PersonDAO personDAO;  
  
    @Autowired  
    public PersonValidator(PersonDAO personDAO) {  
        this.personDAO = personDAO;  
    }  
  
    @Override  
    public boolean supports(Class<?> clazz) {  
        return Person.class.equals(clazz);  
    }  
  
    @Override  
    public void validate(Object target, Errors errors) {  
        Person person = (Person) target;  
  
        if (personDAO.show(person.getEmail()).isPresent()) {  
            errors.rejectValue("email", "email.exists", "Email already exists");  
        }  
        // нужно посмотреть есть ли человек в таким же email  
    }  
}
```


Далее нужно внедрить этот валидатор в контроллер в котором мы проверяем значение:
```java
private final PersonDAO personDAO;  
private final PersonValidator personValidator;  
  
@Autowired  
public PeopleController(PersonDAO personDAO, PersonValidator personValidator) {  
    this.personDAO = personDAO;  
    this.personValidator = personValidator;  
}
```



Ну а потом с помощью этого объекта можно валидировать там где это нужно:
```java
@PostMapping()  
public String create(@ModelAttribute("person") @Valid Person person,  
                     BindingResult bindingResult) {  
    personValidator.validate(person, bindingResult);  // валидация
  
    if (bindingResult.hasErrors()) {  
        log.warn("Binding error in post method: {}", bindingResult.getAllErrors().toString());  
        return "people/new";  
    }  
  
  
    personDAO.save(person);  
    log.info("Person with name = {} and age = {} created", person.getName(), person.getAge());  
  
    return "redirect:/people";  
}
```










# Валидация паттернов


Для этого используется аннотация `@Pattern()` а в скобках указывается регулярное выражение (паттерн) которому должен соответствовать поле



Пример:

![[Pasted image 20250630150114.png]]









# Выпадающие списки



Используются теги `select` и `option`



![[Pasted image 20250630224906.png]]



![[Pasted image 20250630224931.png]]





### В качестве примера разберем ситуацию когда нужно из выпадающего списка выбрать человека и назначить его админом


Сначала создаем контроллер который будет перенаправлять на страницу с этим выпадающим списком:
```java
@GetMapping()  
public String adminPage(Model model, @ModelAttribute("person") Person person) {  
    model.addAttribute("people", personDAO.index());  
  
    return "adminPage";  
}
```



Далее создадим эту страницу (представление:
```html
<!DOCTYPE html>  
<html lang="ru-RU" xmlns:th="http://www.thymeleaf.org">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">  
    <title>Назначить администратора</title>  
</head>  
<body>  
  
<form th:method="PATCH" th:action="@{/admin/add}">  // форма выбора
    <label for="person">Выберите человека</label>  
    <select th:object="${person}" th:field="*{id}" id="person">  
        <option th:each="person : ${people}" th:value="${person.id}" th:text="${person.name}"></option>  
    </select>  
  
    <input type="submit" value="Назначить админом" />  
</form>  
  
  
</body>  
</html>
```



Ну и далее создаем метод контроллера который принимает эти данные и пока просто для примера выводит id человека в консоль:
```java
@PatchMapping("/add")  
public String makeAdmin(Model model, @ModelAttribute("person") Person person) {  
    System.out.println(person.getId());  
  
    return "redirect:/people";  
}
```


