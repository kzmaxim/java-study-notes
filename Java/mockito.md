


# Test Doubles


![[Pasted image 20250531172517.png]]


Подключить mockito:
```xml
<dependency>  
    <groupId>org.mockito</groupId>  
    <artifactId>mockito-core</artifactId>  
    <version>5.17.0</version>  
    <scope>test</scope>  
</dependency>
```





# Mock


### Часть 1: Основы Mock в Mockito для новичков

Mockito — это популярная библиотека в Java, которая помогает тестировать код, особенно когда ты не хочешь или не можешь использовать реальные объекты (например, базу данных или внешние сервисы). Давай шаг за шагом:

---

#### 1. Что такое Mock?
- **Mock** — это "фейковый" объект, который имитирует поведение реального объекта. Ты создаешь его с помощью Mockito, чтобы заменить что-то сложное или ненадежное в твоем тесте.
- Например, представь, что у тебя есть класс, который отправляет email. Ты не хочешь отправлять настоящий email при каждом тесте — вместо этого создаешь Mock, который притворяется, что отправил письмо, но на самом деле ничего не делает.

---

#### 2. Зачем нужен Mock?
- **Изоляция теста:** Ты тестируешь только одну часть кода (например, свою функцию), а не все зависимости (например, базу данных или API).
- **Контроль поведения:** Ты можешь заставить Mock возвращать нужные данные или выбрасывать ошибки, чтобы проверить, как твой код на это реагирует.
- **Скорость:** Тесты с Mock работают быстрее, чем с реальными объектами, потому что не обращаются к внешним системам.

---

#### 3. Как создать Mock?
В Mockito это делается с помощью аннотации `@Mock` или метода `mock()`. Вот простой пример:

```java
import org.mockito.Mockito;
import static org.mockito.Mockito.*;

public class ExampleTest {
    // Создаем Mock объект
    MyService myService = mock(MyService.class);

    // Или с аннотацией (нужно добавить @RunWith(MockitoJUnitRunner.class) или MockitoAnnotations.initMocks(this))
    @Mock
    MyService myServiceWithAnnotation;

    // Пример класса, который ты тестируешь
    public class MyService {
        public String getData() {
            return "Real data";
        }
    }
}
```

- Здесь `MyService` — это класс, который ты хочешь заменить на Mock. Метод `mock()` или аннотация `@Mock` создают фейковый объект.

---

#### 4. Как настроить поведение Mock?
После создания Mock ты можешь сказать, что он должен делать. Например:

```java
// Указываем, что метод getData должен возвращать "Fake data"
when(myService.getData()).thenReturn("Fake data");

// Или выбрасывать исключение
when(myService.getData()).thenThrow(new RuntimeException("Error!"));

// Проверка, что метод был вызван
verify(myService).getData();
```

- `when(...).thenReturn(...)` — задает, что возвращать, когда вызывают метод.
- `verify(...)` — проверяет, вызывали ли метод (и сколько раз).

---

### Часть 2: Практический пример и завершение

Поскольку объяснение может быть длинным, продолжу во второй части. Давай теперь посмотрим, как это работает на примере, и разберем типичные случаи.

---

#### 5. Простой пример теста
Допустим, у тебя есть класс `Calculator`, который зависит от `MathService`. Ты хочешь протестировать `Calculator`, но не хочешь использовать реальный `MathService`.

```java
import static org.mockito.Mockito.*;

public class CalculatorTest {
    @Mock
    MathService mathService;

    Calculator calculator;

    @BeforeEach
    public void setUp() {
        MockitoAnnotations.initMocks(this); // Инициализируем Mock
        calculator = new Calculator(mathService); // Передаем Mock в конструктор
    }

    @Test
    public void testAdd() {
        // Настраиваем Mock: метод add возвращает 5
        when(mathService.add(2, 3)).thenReturn(5);

        // Вызываем метод тестового класса
        int result = calculator.addNumbers(2, 3);

        // Проверяем результат
        assertEquals(5, result);

        // Проверяем, что метод add был вызван
        verify(mathService).add(2, 3);
    }
}

class Calculator {
    private MathService mathService;

    public Calculator(MathService mathService) {
        this.mathService = mathService;
    }

    public int addNumbers(int a, int b) {
        return mathService.add(a, b);
    }
}

class MathService {
    public int add(int a, int b) {
        return a + b; // Реальная логика
    }
}
```

- **Что происходит:**
  - Мы создаем Mock для `MathService`.
  - Говорим Mock, что `add(2, 3)` должен вернуть 5 (хоть реальный метод вернул бы 5, мы контролируем это).
  - Тест проверяет, что `Calculator` правильно использует `MathService`.
  - `verify` убеждается, что метод был вызван с правильными аргументами.

---

#### 6. Типичные случаи использования
- **Когда реальный объект недоступен:** Например, ты не подключен к базе данных.
- **Когда реальный объект медленный:** Например, API-запросы.
- **Когда нужно проверить ошибки:** Mock может симулировать сбой, чтобы увидеть, как твой код обрабатывает исключения.

---

#### 7. Советы для новичков
- Начни с простого: создай Mock и настрой возврат одного значения.
- Используй аннотации `@Mock` и `@InjectMocks` для удобства (последняя автоматически внедряет Mock в тестовый класс).
- Читай ошибки теста: Mockito обычно дает подсказки, если что-то пошло не так.
- Практикуйся: Попробуй написать тесты для своего кода с Mock.

---





# Spy


### Часть 1: Основы Spy в Mockito для новичков

Spy — это еще один полезный инструмент в Mockito, который помогает в тестировании, но он немного отличается от Mock. Я объясню, что это, зачем нужно, и покажу примеры. Если будет длинно, разделю на две части.

---

#### 1. Что такое Spy?
- **Spy** — это "шпион", который оборачивает **реальный объект**. В отличие от Mock, который полностью фейковый и ничего не делает, пока ты не настроишь его поведение, Spy вызывает **реальные методы** объекта, если ты явно не переопределил их поведение.
- Другими словами, Spy — это реальный объект, но ты можешь "подглядывать" за его вызовами или частично изменять его поведение.

---

#### 2. Чем отличается Spy от Mock?
- **Mock**: Полностью фейковый объект. Если ты не настроил поведение (например, `when(...).thenReturn(...)`), он ничего не делает (возвращает `null`, 0 или false).
- **Spy**: Реальный объект. Если ты не указал поведение, он вызывает настоящие методы объекта.
- Пример: Если у тебя есть метод `calculate()` в классе, Mock не сделает ничего, пока ты не скажешь, что возвращать. Spy вызовет настоящий `calculate()` и вернет реальный результат.

---

#### 3. Зачем нужен Spy?
- **Частичное тестирование:** Ты хочешь использовать реальную логику объекта, но иногда подменять поведение отдельных методов.
- **Проверка вызовов:** Ты можешь следить, как твой код взаимодействует с объектом (например, сколько раз вызвали метод).
- **Когда объект сложный:** Если у объекта много методов, и ты не хочешь настраивать поведение для каждого (как в Mock), Spy позволяет оставить реальную логику там, где она не мешает.

---

#### 4. Как создать Spy?
Есть два способа создать Spy в Mockito:
- С помощью метода `spy()`:
  ```java
  MyService myService = new MyService();
  MyService spyService = spy(myService);
  ```
- Или с аннотацией `@Spy` (нужно инициализировать с помощью `MockitoAnnotations.openMocks(this)`):
  ```java
  @Spy
  MyService spyService = new MyService();
  ```

---

#### 5. Простой пример: Создание и настройка Spy
Давай возьмем класс `Calculator`, который выполняет простые вычисления, и создадим для него Spy.

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.MockitoAnnotations;
import org.mockito.Spy;
import static org.mockito.Mockito.*;

public class CalculatorTest {
    @Spy
    Calculator calculator = new Calculator();

    @BeforeEach
    public void setUp() {
        MockitoAnnotations.openMocks(this); // Инициализация Spy
    }

    @Test
    public void testSpy() {
        // Spy вызывает реальный метод add
        int result = calculator.add(2, 3);
        System.out.println(result); // Выведет 5, потому что реальный метод сложил 2 + 3

        // Переопределяем поведение для метода subtract
        when(calculator.subtract(5, 3)).thenReturn(10); // Вместо 2 вернем 10

        // Проверяем переопределенное поведение
        int subtractResult = calculator.subtract(5, 3);
        System.out.println(subtractResult); // Выведет 10, а не 2

        // Проверяем вызовы
        verify(calculator).add(2, 3);
        verify(calculator).subtract(5, 3);
    }
}

class Calculator {
    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }
}
```

- **Что происходит:**
  - Мы создали Spy для `Calculator`.
  - Метод `add` работает как настоящий (2 + 3 = 5).
  - Метод `subtract` мы переопределили с помощью `when(...).thenReturn(...)`, и он вернул 10 вместо 2.
  - `verify` проверил, что оба метода были вызваны.

---

### Часть 2: Более сложный пример и советы

Продолжим с более реалистичным примером и разберем дополнительные детали.

---

#### 6. Реальный пример: Spy для списка
Давай представим, что ты работаешь с `List` (например, `ArrayList`), и тебе нужно протестировать код, который добавляет элементы в список. Ты хочешь использовать реальный `ArrayList`, но иногда подменять поведение.

```java
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.MockitoAnnotations;
import org.mockito.Spy;
import java.util.ArrayList;
import java.util.List;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.*;

public class ListTest {
    @Spy
    List<String> spyList = new ArrayList<>();

    @BeforeEach
    public void setUp() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    public void testSpyList() {
        // Добавляем элементы — реальная логика ArrayList
        spyList.add("One");
        spyList.add("Two");

        // Проверяем, что элементы добавлены
        assertEquals(2, spyList.size());
        assertEquals("One", spyList.get(0));

        // Переопределяем поведение метода get
        when(spyList.get(1)).thenReturn("Fake Two");

        // Проверяем
        assertEquals("Fake Two", spyList.get(1)); // Вернет "Fake Two", а не "Two"

        // Проверяем вызовы
        verify(spyList).add("One");
        verify(spyList).add("Two");
        verify(spyList, times(2)).get(anyInt());
    }
}
```

- **Что происходит:**
  - Мы создали Spy для `ArrayList`.
  - Методы `add` и `size` работают как в настоящем `ArrayList`.
  - Метод `get(1)` мы переопределили, чтобы он возвращал "Fake Two".
  - `verify` проверил, что методы `add` и `get` были вызваны.

---

#### 7. Когда использовать Spy?
- **Когда нужно реальное поведение:** Если объект сложный, и ты не хочешь вручную настраивать поведение всех методов (как в Mock).
- **Когда нужно частично подменить логику:** Например, ты хочешь, чтобы 90% методов работали как обычно, но один метод вел себя по-другому.
- **Для проверки взаимодействий:** Spy позволяет следить за вызовами методов, как и Mock.

---

#### 8. Важные моменты и советы
- **Осторожно с реальной логикой:** Spy вызывает реальные методы, которые могут обращаться к базе данных, сети и т.д. Это может замедлить тесты или вызвать ошибки. Если тебе не нужна реальная логика, лучше использовать Mock.
- **Используй `doReturn` вместо `when` для Spy:** Иногда `when(spy.method()).thenReturn(...)` вызывает реальный метод `method()`, что может быть нежелательно. Вместо этого используй:
  ```java
  doReturn("Fake").when(spy).method();
  ```
- **Не злоупотребляй Spy:** Если ты переопределяешь большинство методов, возможно, тебе лучше использовать Mock.

---

#### 9. Типичные ошибки новичков
- **Забыли, что Spy вызывает реальные методы:** Если метод делает что-то сложное (например, обращается к базе данных), это может сломать тест.
- **Путаница с `when`:** Если метод имеет побочные эффекты, лучше используй `doReturn`.
- **Слишком много Spy:** Если ты используешь Spy только для проверки вызовов, подумай, не лучше ли использовать Mock.

---

#### 10. Практика
Попробуй создать Spy для своего класса:
- Создай объект и оберните его в Spy.
- Вызови несколько методов и проверь, что они работают как настоящие.
- Переопредели поведение одного метода с помощью `doReturn`.
- Используй `verify`, чтобы проверить вызовы.








# Mockito Extension


Подключаем необходимые зависимости:
```xml
<dependency>  
    <groupId>org.mockito</groupId>  
    <artifactId>mockito-junit-jupiter</artifactId>  
    <version>5.17.0</version>  
    <scope>test</scope>  
</dependency>
```



Подключение:
```java
@ExtendWith({  
                MockitoExtension.class  
})
```


### Часть 1: Основы Mockito Extension в JUnit 5 для новичков

Mockito Extension — это специальное расширение для JUnit 5, которое упрощает использование библиотеки Mockito в тестах. Я объясню, что это, зачем нужно, и покажу примеры. Если будет длинно, разделю на две части.

---

#### 1. Что такое Mockito Extension?
- **Mockito Extension** — это инструмент, который "подключает" Mockito к JUnit 5, чтобы ты мог легко использовать аннотации вроде `@Mock`, `@Spy` или `@InjectMocks` без лишнего кода.
- В JUnit 5 есть механизм расширений (Extension Model), и Mockito предоставляет свое расширение (`MockitoExtension`), чтобы автоматизировать создание Mock и Spy.

---

#### 2. Зачем нужен Mockito Extension?
Без Mockito Extension тебе пришлось бы вручную инициализировать Mock и Spy, например, вызывая `MockitoAnnotations.openMocks(this)` в каждом тесте. Extension делает это за тебя, чтобы ты мог сосредоточиться на написании тестов.

- **Упрощает код:** Не нужно вручную инициализировать аннотации.
- **Делает тесты чище:** Ты просто добавляешь аннотации вроде `@Mock`, и все работает.
- **Автоматизирует работу:** Поддерживает создание Mock, Spy и внедрение зависимостей через `@InjectMocks`.

---

#### 3. Как использовать Mockito Extension?
Чтобы использовать Mockito Extension, нужно:
1. Добавить зависимость Mockito для JUnit 5 в твой проект (если ты используешь Maven или Gradle).
2. Подключить расширение в тестовом классе с помощью аннотации `@ExtendWith`.

**Добавление зависимости (Maven):**
Если ты используешь Maven, добавь в `pom.xml`:
```xml
<dependency>  
    <groupId>org.mockito</groupId>  
    <artifactId>mockito-junit-jupiter</artifactId>  
    <version>5.17.0</version>  
    <scope>test</scope>  
</dependency>
```

**Пример теста с Mockito Extension:**
```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.Mock;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class) // Подключаем Mockito Extension
public class CalculatorTest {
    @Mock
    MathService mathService; // Создаем Mock

    Calculator calculator;

    @Test
    public void testAdd() {
        // Создаем объект Calculator, внедряя Mock
        calculator = new Calculator(mathService);

        // Настраиваем поведение Mock
        when(mathService.add(2, 3)).thenReturn(5);

        // Вызываем метод
        int result = calculator.addNumbers(2, 3);

        // Проверяем результат
        assertEquals(5, result);

        // Проверяем вызов
        verify(mathService).add(2, 3);
    }
}

class Calculator {
    private MathService mathService;

    public Calculator(MathService mathService) {
        this.mathService = mathService;
    }

    public int addNumbers(int a, int b) {
        return mathService.add(a, b);
    }
}

class MathService {
    public int add(int a, int b) {
        return a + b;
    }
}
```

- **Что происходит:**
  - Аннотация `@ExtendWith(MockitoExtension.class)` говорит JUnit 5, что нужно использовать Mockito Extension.
  - `@Mock` создает Mock для `MathService` автоматически.
  - Мы используем Mock, как обычно, с `when` и `verify`.

---

#### 4. Что делает Mockito Extension под капотом?
- **Автоматически инициализирует аннотации:** Вместо вызова `MockitoAnnotations.openMocks(this)` вручную, Extension делает это за тебя.
- **Создает Mock и Spy:** Все объекты с аннотациями `@Mock` и `@Spy` создаются автоматически.
- **Внедряет зависимости:** Если ты используешь `@InjectMocks`, Extension сам внедрит Mock или Spy в тестовый класс.

---

### Часть 2: Пример с `@InjectMocks`, `@Spy` и советы

Продолжим с более сложным примером и разберем дополнительные детали.

---

#### 5. Пример с `@InjectMocks` и `@Spy`
Давай добавим в тест `@InjectMocks` и `@Spy`, чтобы показать, как Mockito Extension упрощает работу.

```java
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Spy;
import org.mockito.junit.jupiter.MockitoExtension;
import java.util.ArrayList;
import java.util.List;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
public class ServiceTest {
    @Mock
    Database database; // Mock для базы данных

    @Spy
    List<String> logList = new ArrayList<>(); // Spy для списка логов

    @InjectMocks
    MyService myService; // Внедряем зависимости в MyService

    @Test
    public void testService() {
        // Настраиваем Mock
        when(database.getData()).thenReturn("Mocked Data");

        // Вызываем метод сервиса
        String result = myService.processData();

        // Проверяем результат
        assertEquals("Processed: Mocked Data", result);

        // Проверяем, что в лог добавилась запись (реальная логика Spy)
        assertEquals(1, logList.size());
        assertEquals("Data processed", logList.get(0));

        // Проверяем вызовы
        verify(database).getData();
        verify(logList).add("Data processed");
    }
}

class MyService {
    private Database database;
    private List<String> logList;

    public MyService(Database database, List<String> logList) {
        this.database = database;
        this.logList = logList;
    }

    public String processData() {
        String data = database.getData();
        logList.add("Data processed");
        return "Processed: " + data;
    }
}

class Database {
    public String getData() {
        return "Real Data";
    }
}
```

- **Что происходит:**
  - `@Mock` создает фейковый `Database`.
  - `@Spy` создает шпион для `ArrayList`, который ведет себя как настоящий список, но мы можем следить за его вызовами.
  - `@InjectMocks` автоматически внедряет `database` и `logList` в `MyService`.
  - Мы настраиваем Mock, чтобы `database.getData()` возвращал "Mocked Data".
  - `logList` работает как настоящий список, но мы проверяем его вызовы с помощью `verify`.

---

#### 6. Когда использовать Mockito Extension?
- **Всегда, когда используешь JUnit 5 и Mockito:** Extension делает тесты чище и удобнее.
- **Если у тебя много Mock или Spy:** Extension экономит время на инициализации.
- **Если ты используешь `@InjectMocks`:** Без Extension внедрение зависимостей не сработает автоматически.

---

#### 7. Советы для новичков
- **Убедись, что зависимость добавлена:** Без `mockito-junit-jupiter` Extension не сработает.
- **Используй `@InjectMocks` для сложных классов:** Если твой класс имеет много зависимостей, это упростит создание тестового объекта.
- **Не забывай про `@ExtendWith`:** Без этой аннотации Mockito Extension не подключится.

---

#### 8. Типичные ошибки
- **Забыли `@ExtendWith`:** Аннотации `@Mock` и `@Spy` не сработают, и ты получишь `NullPointerException`.
- **Неправильная версия Mockito:** Убедись, что ты используешь `mockito-junit-jupiter`, а не старую версию для JUnit 4.
- **Зависимости не внедряются:** Если `@InjectMocks` не работает, проверь, что конструктор или поля в твоем классе правильно настроены.

---

#### 9. Практика
Попробуй написать тест с Mockito Extension:
- Создай класс с зависимостью (например, сервис, который использует базу данных).
- Напиши тест, используя `@Mock` для зависимости и `@InjectMocks` для сервиса.
- Настрой поведение Mock и проверь вызовы с помощью `verify`.




То есть эта запись:

```java
@InjectMocks  
private UserService userService;  
@Mock  
private UserDao userDao;
```

Заменяет эту запись:

```java
this.userDao = Mockito.mock(UserDao.class);  
this.userService = new UserService(userDao);
```








# Behavior Driven Development


Основные понятия: given --- when --- then


![[Pasted image 20250531235505.png]]




