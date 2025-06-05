# JUNIT5

[[mockito|Mockito]]



Подключить Junit5 через Maven:
```xml
<!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-slf4j-impl -->  
<dependency>  
    <groupId>org.apache.logging.log4j</groupId>  
    <artifactId>log4j-slf4j-impl</artifactId>  
    <version>2.24.3</version>  
    <scope>compile</scope>  
</dependency>
```


Допустим есть какой-то класс и в нем методы. Чтобы создать тесты, нужно создать отдельный класс который называется `Имя класса + Test`, а также к каждому методу который тестирует использовать аннотацию `@Test`:

```java
class CalculatorTest {
   	@Test
    	public void add() {
    	}
   	@Test
    	public void sub() {
        }
   	@Test
    	public void mul() {
    	}
   	@Test
    	public void div() {
    	}
}
```


## Примеры тестов Junit
```java
class CalculatorTest {
   	@Test
    	public void add() {
        	Calculator calc = new Calculator();
        	int result = calc.add(2, 3);
        	assertEquals(5, result);
    	}

   	@Test
    	public void sub() {
        	Calculator calc = new Calculator();
        	int result = calc.sub(10, 10);
        	assertEquals(0, result);
        }

   	@Test
    	public void mul() {
        	Calculator calc = new Calculator();
        	int result = calc.mul(-5, -3);
        	assertEquals(15, result);
    	}

   	@Test
    	public void div() {
        	Calculator calc = new Calculator();
        	int result = calc.div(2, 3);
        	assertEquals(0, result);
    	}
}
```


Вот так выглядит типичный тест с использованием JUnit. Из интересного: тут используется специальный метод `assertEquals()`: он сравнивает переданные ему параметры, о чем говорит слово equals в его названии.

Если параметры, переданные в `assertEquals()`, неравны, то метод кинет исключение и тест будет считаться проваленным. Это исключение не помешает выполнению остальных тестов.


Чтобы каждый раз в тестовом методе не писать создание объекта, его можно вынести в отдельный метод и поставить ему специальную аннотацию `@BeforeEach`. Тогда Junit будет вызывать этот метод перед каждым тестовым методом.
Пример:
```java
class HttpClientTest {
   	public HttpClient client;

 	@BeforeEach
  	public void init(){
 	   client = HttpClient.newBuilder()
 	        .version(Version.HTTP_1_1)
 	        .followRedirects(Redirect.NORMAL)
 	        .connectTimeout(Duration.ofSeconds(20))
 	        .proxy(ProxySelector.of(new InetSocketAddress("proxy.example.com", 80)))
 	        .authenticator(Authenticator.getDefault())
 	        .build();
  	}

 	@Test
  	public void send200() throws Exception {
   	   //Создаем объект HttpRequst()
       	HttpRequest request = HttpRequest.newBuilder(new URI("https://javarush.com")).build();

   	   //Вызываем метод send()
   	   HttpResponse


  response = client.send(request, BodyHandlers.ofString()); assertEquals(200, response.statusCode()); } @Test public void send404() throws Exception { //Создаем объект HttpRequst() HttpRequest request = HttpRequest.newBuilder(new URI("https://javarush.com/unknown")).build(); //Вызываем метод send() HttpResponse
 
   response = client.send(request, BodyHandlers.ofString()); assertEquals(404, response.statusCode()); } }
 
```

Если у тебя есть 3 тестовых метода test1(), test2() и test3(), то порядок вызова будет таким:

- BeforeEach-метод
- test1()
- AfterEach-метод
- BeforeEach-метод
- test2()
- AfterEach-метод
- BeforeEach-метод
- test3()
- AfterEach-метод


JUnit также позволяет добавить метод, который будет вызван один раз перед всеми тестовыми методами. Такой метод нужно пометить аннотацией `@BeforeAll`. Для нее так же существует парная аннотация `@AfterAll`. Метод, помеченный ею, JUnit вызовет после всех тестовых методов.


## Полезные аннотации в JUnit

- `@Disabled` - отключает тест
- `@Nested` - позволяет тестировать методы во вложенных классах. Пример:
```java
public class AppTest {
    @Nested
    public class DevStagingEnvironment {
    @Test
        void testOnDev(){
            System.setProperty("ENV", "DEV");
            Assumptions.assumeFalse("DEV".equals(System.getProperty("ENV")));
        }
   }

    @Nested
    public class ProductionEnvironment {
        @Test
        void testOnProd(){
           System.setProperty("ENV", "PROD");
           Assumptions.assumeFalse("DEV".equals(System.getProperty("ENV")));
        }
   }
}
```
- `@ExtendWith` - используется для **подключения расширений (extensions)** — специальных классов, которые могут **расширять поведение тестов**: например, выполнять код до/после теста, внедрять зависимости, мокать объекты и т.д.
```java
@ExtendWith(ИмяРасширения.class)
class MyTest {
    // тесты
}
```

- `@Timeout` - позволяет задать время на выполнение теста

- `ParameterizedTest` - используется для **запуска одного и того же теста с разными наборами данных**. Это удобно, когда ты хочешь проверить поведение метода на разных входных значениях — без дублирования кода.
Для использования параметризованных тестов надо добавить еще одну зависимость в ваш `pom.xml`:

```
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-params</artifactId>
    <version>5.8.2</version>
    <scope>test</scope>
</dependency>
```

После чего можем рассмотреть пример:
```java
@ParameterizedTest
@ValueSource(ints = { 1, 2, 3 })
void testMethod(int argument) {
    //test code
}

@ParameterizedTest
@ValueSource(ints = { 1, 2, 3 })
void testMethodWithAutoboxing(Integer argument) {
    //test code
}
```
Каждый тестовый метод вызовется по 3 раза и при каждом вызове в него будет передан очередной параметр. Значения задаются с помощью другой аннотации — `@ValueSource`. Но о ней нужно рассказать подробнее.


## Аннотация @ValueSource

Аннотация @ValueSource отлично подходит для работы с примитивами и литералами. Просто перечисли значения параметра через запятую и тест будет вызван по одному разу для каждого значения.

```java
@ParameterizedTest
@ValueSource(ints = { 1, 2, 3 })
void testMethodWithAutoboxing(Integer argument) {
    //test code
}
```

Но есть и минус — эта аннотация не поддерживает null, так что для него нужно будет использовать специальный костыль — @NullSource. Выглядит это так:

```java
@ParameterizedTest
@NullSource
void testMethodNullSource(Integer argument) {
    assertTrue(argument == null);
}
```


- `@EnumSource` - передает в метод все значения определенного Enum’а. Используется в `@ParameterizedTest`
- `@MethodSource`- А как же передавать в качестве параметров объекты? Особенно, если они имеют сложный алгоритм построения. Для этого можно просто указать специальный вспомогательный метод, который будет возвращать список (Stream) из таких значений.

Пример:
```java
@ParameterizedTest
@MethodSource("argsProviderFactory")
void testWithMethodSource(String argument) {
    assertNotNull(argument);
}

static Stream<String> argsProviderFactory() {
    return Stream.of("один", "два",  "три");
}
```
Конечно вы уже задались вопросом, а что делать, если в метод хочется передать несколько параметров. Для этого есть очень классная аннотация @CSVSource. Она позволяет перечислить значения параметров метода просто через запятую.

Пример:
```java
@ParameterizedTest
@CsvSource({
    "alex, 30, Программист, Работает",
    "brian, 35, Тестировщик, Работает",
	"charles, 40, Менеджер, Пинает"
})
void testWithCsvSource(String name, int age, String occupation, String status) {
	//код метода
}
```




  
## JUnit Assertions


Ассерты (asserts) — это специальные проверки, которые можно вставить в разные места кода. Их задача определять, что что-то пошло не так. Вернее, проверять, что все идет как нужно. Вот это “как нужно” они и позволяют задать различными способами.

```java
assertEquals(эталон, значение)
```

Ниже я приведу список самых популярных методов — ассертов. По их названиям вполне можно догадаться как они работают. Но все же напишу краткое пояснение:

|   |   |
|---|---|
|assertEquals|Проверяет, что два объекта равны|
|assertArrayEquals|Проверяет, что два массива содержат равные значения|
|assertNotNull|Проверяет, что аргумент не равен null|
|assertNull|Проверяет, что аргумент равен null|
|assertNotSame|Проверят, что два аргумента — это не один и тот же объект|
|assertSame|Проверят, что два аргумента — это один и тот же объект|
|assertTrue|Проверяет, что аргумент равен _true_|
|assertFalse|Проверяет, что аргумент равен _false_|


Но если один из параметров не совпадет, то проверки остальных не произойдет. А хотелось бы чтобы они все-таки произошли и их результаты записались в лог. Но при этом, если хотя бы одна проверка не прошла, то тест все-таки был провален.

Для этого есть специальный метод — assertAll(). В качестве первого аргумента он принимает комментарий, который нужно записать в лог, а дальше — любое количество функций-ассертов.

Вот как будет переписан наш пример с его помощью:
```java
Address address = unitUnderTest.methodUnderTest();
assertAll("Сложный сценарий сравнение адреса",
    () -> assertEquals("Вашингтон", address.getCity()),
    () -> assertEquals("Oracle Parkway", address.getStreet()),
    () -> assertEquals("500", address.getNumber())
);
```



Очень часто бывают ситуации, когда тебе нужно убедиться, что в определенной ситуации код кидает нужное исключение: определил ошибку и кинул нужное исключение. Это очень распространенная ситуация.

На этот случай есть еще один полезный метод assert — это assertThrows(). Общий формат его вызова имеет вид:
```java
assertThrows(исключение, код)
```







## Уровни тестирования:

1. `Unit testing` - тестирование маленького компонента приложения (функции)
2. `Integration testing` - тестирование нескольких компонентов(функций), т.е. как маленькие units работают вместе как один целый unit
3. `Acceptance testing` - тестирование приложения в целом, т.е как оно работает со стороны пользователя (функциональное тестирование)



## Подключить junit

```xml
<dependency>  
    <groupId>org.junit.jupiter</groupId>  
    <artifactId>junit-jupiter-engine</artifactId>  
    <version>5.11.4</version>  
    <scope>test</scope>  
</dependency>
```


## Установить wrapper
```shell
mvn -N io.takari:maven:wrapper -Dmaven=3.9.9
```









# Аннотация @Test. Assertions


**Пример теста:**
```java
@Test  
void usersEmptyIfNoUserAdded() {  
    UserService userService = new UserService();  
    List<User> users = userService.getAll();  
    assertTrue(users.isEmpty());  
}
```


- `assertTrue()` - проверяет является ли выражение в скобках true

Чтобы при фэйле выводилось какое-нибудь сообщение, нужно у метода указать строку вторым параметром:
```java
assertTrue(users.isEmpty(), "Users list should not be empty");
```



Есть еще методы `assertEquals()`, `assertSame()`, `assertArrayEquals()`, `assertAll()`




# Test Lifecycle


![[Pasted image 20250528190512.png]]


### @BeforeEach
Вызывается перед каждым тестом

Пример:
```java
private UserService userService;  
  
@BeforeEach  
void prepare() {  
    System.out.println("Before each: " + this);  
    userService = new UserService();  
}
```


### @AfterEach
Вызывается после каждого теста

Пример:
```java
@AfterEach  
void deleteDataFromDatabase() {  
    System.out.println("After each: " + this);  
}
```


### @BeforeAll
Вызывается один раз перед всеми тестами

Пример:
```java
@BeforeAll  
static void init() {  
    System.out.println("Before all: ");  
}
```

Метод должен обязательно быть статическим



### AfterAll
Вызывается один раз после всех тестов

Пример:
```java
@AfterAll  
static void cleanUp() {  
    System.out.println("After all: ");  
}
```







# Запуск тестов. Launcher API


![[Pasted image 20250528192559.png]]


Подключить launcher:
```xml
<dependency>  
    <groupId>org.junit.platform</groupId>  
    <artifactId>junit-platform-launcher</artifactId>  
    <version>1.12.1</version>  
    <scope>test</scope>  
</dependency>
```


Пример лаунчера:
```java
public class TestLauncher {  
    public static void main(String[] args) {  
        Launcher launcher = LauncherFactory.create();  
  
        SummaryGeneratingListener summaryGeneratingListener = new SummaryGeneratingListener();  
  
  
        LauncherDiscoveryRequest request = LauncherDiscoveryRequestBuilder  
                .request()  
                .build();  
  
        launcher.execute(request, summaryGeneratingListener);  
    }  
}
```





# Test Driven Development. TDD


![[Pasted image 20250528193918.png]]


То есть это когда мы сначала пишем тест для функциональности, а потом уже саму функциональность.

Пример, пишем сначала тесты:

```java
@Test  
void loginSuccessIfUserExists() {  
    userService.add(ALEX);  
    Optional<User> maybeUser = userService.login(ALEX.getUsername(), ALEX.getPassword());  
  
    assertTrue(maybeUser.isPresent());  
    maybeUser.ifPresent(user -> assertEquals(ALEX, user));  
}
```

```java
@Test  
void loginFailIfPasswordIsNotCorrect() {  
    userService.add(ALEX);  
    Optional<User> maybeUser = userService.login(ALEX.getUsername(), 4433L);  
  
    assertTrue(maybeUser.isEmpty());  
}
```

```java
@Test  
void loginFailIfUserDoesNotExist() {  
    userService.add(ALEX);  
    Optional<User> maybeUser = userService.login("hello", 4433L);  
  
    assertTrue(maybeUser.isEmpty());  
}
```



А потом реализуем функциональность:

User:
```java
@Value(staticConstructor = "of")  
public class User {  
    Integer id;  
    String username;  
    Long password;  
}
```


Метод login:
```java
public Optional<User> login(String username, Long password) {  
    return users.stream()  
            .filter(user -> user.getPassword().equals(password))  
            .filter(user -> user.getUsername().equalsIgnoreCase(username))  
            .findFirst();  
}
```






# AssertJ & Hamcrest




## AssertJ

Подключение AssertJ:
```xml
<dependency>  
    <groupId>org.assertj</groupId>  
    <artifactId>assertj-core</artifactId>  
    <version>3.26.3</version>  
    <scope>test</scope>  
</dependency>
```



Пример:
```java
Assertions.assertThat(users).hasSize(2);  // AssertJ

assertEquals(2, users.size());  // Junit
```


Или так:
```java
assertThat(maybeUser).isPresent();  // AssertJ

assertTrue(maybeUser.isPresent());  // Junit
```







# Testing Exceptions


Пример как можно проверять на выбрасываемые исключения:
```java
@Test  
void throwExceptionIfUsernameOrPasswordIsNull() {  
    try {  
        userService.login(null, 12234L);  
        fail("method should throw exception on username is null");  
    } catch(IllegalArgumentException e) {  
        assertTrue(true);  
    }  
}
```


Но чтобы не писать такой код, можно использовать ассерт `assertThrows`:
```java
@Test  
void throwExceptionIfUsernameOrPasswordIsNull() {  
    assertThrows(IllegalArgumentException.class, () -> userService.login(null, 12234L));  
}
```


Но в этом методе проверяется только один случай, вот как можно проверить сразу два случая:
```java
@Test  
void throwExceptionIfUsernameOrPasswordIsNull() {  
    assertAll(  
            () -> assertThrows(IllegalArgumentException.class, () -> userService.login(null, 12234L)),  
            () -> assertThrows(IllegalArgumentException.class, () -> userService.login("Name", null))  
    );  
}
```




# Tagging and Filtering


Используется аннотация `@Tag`

Например можно пометить методы которые относятся к логину:
```java
@Test  
    @Tag("login")  
    void loginSuccessIfUserExists() {  
        userService.add(ALEX);  
        Optional<User> maybeUser = userService.login(ALEX.getUsername(), ALEX.getPassword());  
  
  
        assertThat(maybeUser).isPresent();  
//        assertTrue(maybeUser.isPresent());  
        maybeUser.ifPresent(user -> assertEquals(ALEX, user));  
    }
```
```java
@Test  
@Tag("login")  
void loginFailIfPasswordIsNotCorrect() {  
    userService.add(ALEX);  
    Optional<User> maybeUser = userService.login(ALEX.getUsername(), 4433L);  
  
    assertTrue(maybeUser.isEmpty());  
}  
  
@Test  
@Tag("login")  
void loginFailIfUserDoesNotExist() {  
    userService.add(ALEX);  
    Optional<User> maybeUser = userService.login("hello", 4433L);  
  
    assertTrue(maybeUser.isEmpty());  
}
```









# Tests order. Nested tests


Используется аннотация `@TestMethodOrder()`. В скобки нужно передать класс наследующийся от `MethodOrderer`

- `MethodOrderer.Random` - все тесты выполняются рандомно
- `MethodOrderer.OrderAnnotation` - задается порядок. Чтобы задать порядок нужно помечать тесты аннотацией `@Order()` а в скобках писать число порядка
- `MethodOrderer.MethodName` - тесты будут вызываться по названию в алфавитном порядке
- `MethodOrderer.DisplayName` - сортируется по display name а оно задается при помощи аннотации `@DisplayName`



Для разграничения функционала можно использовать Nested tests, то есть создать вложенный класс и поместить в него тесты. Например создать класс LoginTest и поместить в него тесты которые проверяют на login. Чтобы это работало нужно пометить этот класс аннотацией `@Nested`









# Parameterized tests


Для начала нужно подключить зависимость:
```xml
<dependency>  
    <groupId>org.junit.jupiter</groupId>  
    <artifactId>junit-jupiter-params</artifactId>  
    <version>5.11.4</version>  
    <scope>test</scope>  
</dependency>
```


И теперь пишем не `@Test`, а `@ParameterizedTest`




`@NullAndEmptySource` - подставляют в тест пустые значения и null значения, работают только для тестов которые принимают один параметр

`@ValueSource` - аннотация которая подставляет различные значения в параметры


Например:

```java
@ParameterizedTest  
@NullAndEmptySource  // подставит null и пустые значения
@ValueSource(strings = {  
        "Vanya", "Petya"  // подставит эти значения
})  
void loginParameterizedTest(String username) {  
    userService.add(ALEX);  
    userService.add(VANYA);  
    Optional<User> maybeUser = userService.login(username, null);  
}
```


`@MethodSource` - в эту аннотацию нужно передать название метода, который будет подставлять значения в тест.

Например:

Тест:
```java
@ParameterizedTest  
@MethodSource("getArgumentsForLoginTest")  
void loginParameterizedTest(String username, Long password, Optional<User> user) {  
    userService.add(ALEX);  
    userService.add(VANYA);  
    Optional<User> maybeUser = userService.login(username, password);  
    assertThat(maybeUser).isEqualTo(user);  
}
```

Метод:
```java
static Stream<Arguments> getArgumentsForLoginTest() {  
    return Stream.of(  
            Arguments.of("Vanya", 1234L, Optional.of(VANYA)), // подставляем нормальные значения 
            Arguments.of("Alex", 7524L, Optional.of(ALEX)),  
            Arguments.of("Alex", 9994L, Optional.empty()),  // подставляем неправильное значение для пароля
            Arguments.of("Kto-to", 7524L, Optional.empty())  // подставляем неправильно значение для имени
    );  
}
```


Этот формат используется наиболее часто



`@CsvFileSource` - в csv файле пишутся тестовые значения






# Flaky Tests. Timeouts


**Flaky tests** - это нестабильные тесты, то есть иногда они проходят, а иногда падают


`@Disabled` - отключить тест

`@RepeatedTest(value = 5, name= RepeatedTest.LONG_DISPLAY_NAME)` - этот тест будет повторяться 5 раз


Эта аннотация позволяет выявить fluky-тесты, когда на 1-2 итерации все нормально, а на 3 ил 4 тест падает





### Timeout

`assertTimeout()` - установить таймаут

Также можно сделать при помощи аннотации `@Timeout`







# Extenstion model


![[Pasted image 20250531163843.png]]



Используется аннотация `@ExtendWith`:
![[Pasted image 20250531164629.png]]



### 1. Test lifecycle callbacks (Обратные вызовы жизненного цикла теста)

JUnit 5 предоставляет **расширения** (extensions), которые позволяют внедряться в различные этапы жизненного цикла теста. Это означает, что вы можете выполнять определенные действия до или после выполнения теста, набора тестов или всего тестового класса.

- **Примеры расширений жизненного цикла:**
    - BeforeAllCallback — выполняется один раз перед всеми тестами в классе.
    - BeforeEachCallback — выполняется перед каждым тестом.
    - AfterEachCallback — выполняется после каждого теста.
    - AfterAllCallback — выполняется один раз после всех тестов в классе.
- **Пример использования:** Вы можете использовать это для настройки окружения (например, инициализации базы данных) или очистки ресурсов после тестов.
- **Как применить:** Реализуйте интерфейс (например, BeforeEachCallback) и зарегистрируйте расширение через аннотацию @ExtendWith.

---

### 2. Test instance post-processing (Постобработка экземпляров тестов)

Этот механизм позволяет изменять или настраивать экземпляры тестовых классов после их создания, но перед выполнением тестов.

- **Что это значит:** JUnit 5 создает экземпляр тестового класса (по умолчанию для каждого теста — новый экземпляр, если не указано иное). Расширение может модифицировать этот экземпляр.
- **Интерфейс:** TestInstancePostProcessor.
- **Пример:** Можно внедрить зависимости в тестовый класс (например, через DI — Dependency Injection), если вы используете фреймворки вроде Spring или Mockito.
- **Как применить:** Реализуйте TestInstancePostProcessor и укажите расширение через @ExtendWith.

---

### 3. Conditional test execution (Условное выполнение тестов)

Этот пункт относится к возможности управлять выполнением тестов на основе определенных условий.

- **Что это значит:** Вы можете указать, должен ли тест выполняться, в зависимости от внешних факторов (например, операционной системы, переменных окружения или пользовательских условий).
- **Интерфейс:** ExecutionCondition.
- **Пример:**
    - Тест выполняется только на Linux: @EnabledOnOs(OS.LINUX).
    - Тест отключается, если определенная переменная окружения не установлена: @EnabledIfEnvironmentVariable.
- **Как применить:** Реализуйте ExecutionCondition или используйте встроенные аннотации вроде @EnabledIf или @DisabledIf.

---

### 4. Parameter resolution (Разрешение параметров)

Этот механизм позволяет передавать параметры в тестовые методы через расширения.

- **Что это значит:** JUnit 5 может автоматически предоставлять объекты (параметры) в методы тестов, @BeforeEach, @AfterEach и т.д., если расширение реализует логику их создания.
- **Интерфейс:** ParameterResolver.
- **Пример:**
    - Вы можете внедрить объект TestInfo или TestReporter в метод теста.
    - Если вы используете Spring, расширение SpringExtension может внедрять бины (например, сервисы) в тестовые методы.
- **Как применить:** Реализуйте ParameterResolver и укажите расширение через @ExtendWith.

---

### 5. Exception handling (Обработка исключений)

Этот пункт относится к возможности перехватывать и обрабатывать исключения, возникающие во время выполнения тестов.

- **Что это значит:** Расширение может перехватить исключение, выброшенное тестовым методом, и решить, как с ним поступить (например, логировать, игнорировать или пометить тест как проваленный).
- **Интерфейс:** TestExecutionExceptionHandler.
- **Пример:** Вы можете настроить обработку, чтобы тест считался успешным, даже если выброшено определенное исключение, или наоборот — пометить тест как проваленный при неожиданных исключениях.
- **Как применить:** Реализуйте TestExecutionExceptionHandler и зарегистрируйте расширение через @ExtendWith.

---

### Итог

**Extension Model** в JUnit 5 — это мощный инструмент, который позволяет гибко настраивать поведение тестов. Вы можете:

- Управлять жизненным циклом тестов (Test lifecycle callbacks).
- Модифицировать тестовые экземпляры (Test instance post-processing).
- Устанавливать условия выполнения тестов (Conditional test execution).
- Внедрять зависимости в тесты (Parameter resolution).
- Обрабатывать исключения (Exception handling).

Каждый из этих механизмов реализуется через создание пользовательских расширений и регистрацию их с помощью аннотации @ExtendWith. Это делает JUnit 5 очень гибким для сложных сценариев тестирования.




### Test Lifecycle Callbacks

![[Pasted image 20250531165008.png]]


Вот пример класса который внедряется в один или сразу несколько этапов цикла:

```java
public class GlobalExtension implements BeforeAllCallback, AfterTestExecutionCallback {  
  
    @Override  
    public void beforeAll(ExtensionContext context) throws Exception {  
        System.out.println("Before all callback");  
    }  
  
    @Override  
    public void afterTestExecution(ExtensionContext context) throws Exception {  
        System.out.println("After all execution callback");  
    }  
}
```

И вот как совершается подключение:

```java
@ExtendWith(  
        GlobalExtension.class  // подключение
)  
class UserServiceTest {  
    private static final User VANYA = User.of(1, "Vanya", 1234L);  
    private static final User ALEX = User.of(2, "Alex", 7524L);  
    private UserService userService;  
  
    @BeforeAll  
    static void init() {  
        System.out.println("Before all: ");  
    }  
  
    @BeforeEach  
    void prepare() {  
        System.out.println("Before each: " + this);  
        userService = new UserService();  
    }

	...
```

