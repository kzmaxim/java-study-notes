[[hibernate|Hibernate]]

![[Pasted image 20250324184346.png]]



Для начала работы с JDBC необходимо сначала скачать драйвер той базы данных, которую вы будете использовать. В моем случае это Postgresql, поэтому я просто скачал jar-файл и применил его к проекту моему.

![[Pasted image 20250324191553.png]]

Далее через раздел Database я подключился к серверу моей базы данных 


## Подключение к базам данных


Для того чтобы установить соединение с базой данных используется DriverManager:
```java
DriverManager.getConnection(url, username, password);
```

Метод `getConnection` возвращает объект типа `Connection`


`DriverManager` стоит оборачивать в `try-with-resources`:

```java
try(var connection = DriverManager.getConnection(url, username, password)) {  
    System.out.println("Connected to database");  
}
```

Но вообще для подключения к БД создают отдельный класс который это подключение возвращает:

```java
public final class ConnectionManager {  
    private static final String URL = "jdbc:postgresql://localhost:5432/Edme";  
    private static final String USERNAME = "maxim";  
    private static final String PASSWORD = "3maxim14";  
  
    static {  
        try {  
            Class.forName("org.postgresql.Driver");  
        } catch (ClassNotFoundException e) {  
            throw new RuntimeException(e);  
        }  
    }  
  
  
    private ConnectionManager() {}  
  
    public static Connection open() {  
        try {  
            return DriverManager.getConnection(URL, USERNAME, PASSWORD);  
        } catch (SQLException e) {  
            throw new RuntimeException(e);  
        }  
    }  
}
```


И вот так указываем в методе main или там где мы хотим получить соединение:

```java
try(var connection = ConnectionManager.open()) {  
    System.out.println("Connected to database");  
    System.out.println(connection.getTransactionIsolation());  
}
```


Строка
```java
static {  
        try {  
            Class.forName("org.postgresql.Driver");  
        } catch (ClassNotFoundException e) {  
            throw new RuntimeException(e);  
        }  
    } 
```
нужна для версии Java до 1.8, так как до этой версии java могла не найти нужный драйвер



Для хранения url, username и password используется специальный файл .properties
В нем как раз указываются все эти значения в виде ключ-значение
Этот файл следует хранить в директории `resources`
Вот пример того что нужно писать в этом файле:
```
db.url=jdbc:postgresql://localhost:5432/Edme  
db.username=maxim  
db.password=3maxim14
```



Чтобы работать с этим файлом нужно создать утильный класс PropertiesUtil:

```java
public final class PropertiesUtil {  
    private PropertiesUtil() {}  
    private static final Properties PROPERTIES = new Properties();  
      
    static {  
        loadProperties();  
    }  
    public static String get(String key) {  
        return PROPERTIES.getProperty(key);  
    }  
    private static void loadProperties() {  
        try(var inputStream = Properties.class.getClassLoader().getResourceAsStream("application.properties")) {  
            PROPERTIES.load(inputStream);  
        } catch (IOException e) {  
            throw new RuntimeException(e);  
        }  
    }  
}
```

Класс `PropertiesUtil` — это **утилита для работы с `.properties`-файлами** в Java. Он упрощает загрузку и доступ к настройкам, которые хранятся в файле `application.properties`.

### **Что делает `PropertiesUtil`?**

1. **Загружает настройки из файла**
    
    - Читает файл `application.properties` (например, настройки базы данных, API-ключи, пути).
        
    - Сохраняет их в памяти в виде объекта `Properties`.
        
2. **Даёт удобный доступ к настройкам**
    
    - Позволяет получить значение по ключу через метод `get()`.
        
3. **Работает один раз при старте программы**
    
    - Настройки загружаются в статическом блоке инициализации (`static {}`), поэтому файл читается только один раз при первом обращении к классу.


### **Зачем это нужно?**

1. **Отделение конфигурации от кода**
    
    - Параметры (URL БД, пароли) хранятся не в коде, а в файле.
        
    - Можно менять настройки без перекомпиляции.
        
2. **Удобство**
    
    - Не нужно каждый раз открывать файл — настройки загружаются один раз.
        
    - Доступ к любому параметру через `PropertiesUtil.get("ключ")`.
        
3. **Безопасность**
    
    - Файл `application.properties` можно исключить из Git (через `.gitignore`), чтобы не хранить в репозитории пароли/ключи.




И потом нужно поменять метод open() в ConnectionManager:
```java
public final class ConnectionManager {  
// поменяли эти поля и изменили их на ключи
    private static final String URL_KEY = "db.url";  
    private static final String USERNAME_KEY = "db.username";  
    private static final String PASSWORD_KEY = "db.password";  
  
    static {  
        try {  
            Class.forName("org.postgresql.Driver");  
        } catch (ClassNotFoundException e) {  
            throw new RuntimeException(e);  
        }  
    }  
  
  
    private ConnectionManager() {}  
    
  // поменяли метод open()
    public static Connection open() {  
        try {  
            return DriverManager.getConnection(  
                    PropertiesUtil.get(URL_KEY),  
                    PropertiesUtil.get(USERNAME_KEY),  
                    PropertiesUtil.get(PASSWORD_KEY)  
            );  
        } catch (SQLException e) {  
            throw new RuntimeException(e);  
        }  
    }  
}
```





## Statement


`execute(String) boolean` - метод который используется для выполнения в основном SELECT запросов (если это SELECT то метод возвращает true, а если какая-то другая команда то false)

statement помещается в try-with-resources вместе с DriverManager:
```java
try(var connection = ConnectionManager.open();  
var statement = connection.createStatement()) {  
}
```

и создается при помощи метода `createStatement()` у объекта connection
```java
var statement = connection.createStatement();
```

Вот пример выполнения SQL-запроса с помощью метода `execute()`:
```java
String sql = """  
        CREATE TABLE users (id INT, name VARCHAR(255), password VARCHAR(255));  
        """;  
try(var connection = ConnectionManager.open();  
var statement = connection.createStatement()) {  
    var result = statement.execute(sql);  
    System.out.println(result);  
}
```

На выводе будет `false` потому что я выполнил не SELECT запрос

После этого в базе данных была создана таблица users:
![[Pasted image 20250324204311.png]]



`executeUpdate(String) int` - используется для INSERT, UPDATE, DELETE - операций
`executeQuery(String) ResultSet` - используется для SELECT-операций


Вот как работать с SELECT-запросами и методом `executeQuery` :
```java
String sql = """  
        SELECT * FROM info;  
        """;  
try(var connection = ConnectionManager.open();  
var statement = connection.createStatement()) {  
    var result = statement.executeQuery(sql);  
    while(result.next()) {  
        System.out.print(result.getLong("id") + " " + result.getString("data"));  
        System.out.println();  
    }  
}
```
Здесь я использовал метод `executeQuery` и он возвратил объект типа `ResultSet`. У объекта этого типа есть метод `next` - это аналог того же метода но у Iterator. С помощью цикла while я прохожу по каждой строке в таблице и возвращаю id с помощью
```java
result.getLong("id")
```
и data :
```java
result.getString("data")
```


### Generated keys

Если мы хотим чтобы при вставке значений мы также получали auto generated keys то нам в этом поможет метод у стэйтмента `getGeneratedKeys()` :

```java
var result = statement.executeUpdate(sql, Statement.RETURN_GENERATED_KEYS); // указываем флаг чтобы можно было вернуть ключи
ResultSet generatedKeys = statement.getGeneratedKeys();
if (generatedKeys.next()) {
	var generatedId = generatedKeys.getInt("id");
	System.out.println(generatedId);
}
```




### SQL injection


Некоторые поля могут возвратить Null (например методы `getLong`, `getInt`) и поэтому можно использовать метод `getObject`:

```java
res.add(result.getObject("id", Long.class)); // NULL safe
```


А сейчас будет пример SQL injection:

```java
public class Main {  
    public static void main(String[] args) throws SQLException {  
        String infoId = "4";  // здесь может быть sql запрос
        List<String> info = getInfoById(infoId);  
        System.out.println(info);  
    }  
    static List<String> getInfoById(String infoId) throws SQLException {  
        String sql = """  
                SELECT data   
FROM info WHERE id = %s  // сюда может быть передан вредный запрос вместо id
                """.formatted(infoId);  
        List<String> res = new ArrayList<>();  
        try (var connection = ConnectionManager.open();  
        var statement = connection.createStatement()) {  
            var result = statement.executeQuery(sql);  
            while (result.next()) {  
                res.add(result.getObject("data", String.class)); // NULL safe  
            }  
        }  
        return res;  
    }  
}
```

Этот код возвращает данные по переданному id, но что тут не так? Не так тут то что id передается напрямую в строку и вместо id туда можно передать SQL-запрос который может как нибудь навредить нам. Например, он вернет все данные из таблицы или вовсе удалит таблицу.

Для предотвращения SQL injection используется `PreparedStatement`


### PreparedStatement


**Создание `PreparedStatement`** 
```java
var statement = connection.prepareStatement(sql);
```

Как видите при создании стэйтмента нужно передавать sql


Вот исправленный вариант прошлого кода но с использованием `PreparedStatement` и защищенный от Sql injection:

```java
public class Main {  
    public static void main(String[] args) throws SQLException {  
        Long infoId = 4L;  
        List<String> info = getInfoById(infoId);  
        System.out.println(info);  
    }  
    static List<String> getInfoById(Long infoId) throws SQLException {  
        String sql = """  
                SELECT data   
FROM info WHERE id = ?  // вместо форматирования используем '?'
                """;  
        List<String> res = new ArrayList<>();  
        try (var connection = ConnectionManager.open();  
        var statement = connection.prepareStatement(sql)) {  // создаем PreparedStatement
  
            statement.setLong(1, infoId);  // устанавливаем значение вместо вопросика
  
            var result = statement.executeQuery();  // выполняем запрос
            while (result.next()) {  
                res.add(result.getObject("data", String.class)); // NULL safe  
            }  
        }  
        return res;  
    }  
}
```

Теперь наш код защищен от SQL injection



### FetchSize


**FetchSize** - это параметр который означает количество записей, которые мы берем из базы данных и переносим в оперативную память Java-приложения. То есть если у нас 1000 строк мы выбрали с помощью запроса, а fetch size у нас 100, то только 100 строк перенесутся с сервера к Java-приложения. А остальные строки будут подгружаться после того как мы обработаем предыдущие 100 строк.

FetchSize можно установить у стэйтмента при помощи метода `setFetchSize()`:

```java
preparedStatment.setFetchSize(100); // установил FetchSize
```


Есть также метод `setQueryTimeout()` который означает сколько времени нужно ждать выполнения SQL-запроса:

```java
preparedStatement.setQueryTimeout(10);
```

И есть метод `setMaxRows()` который ограничивает количество строк которые мы будем получать с сервера:

```java
preparedStatement.setMaxRows(100);
```




### MetaData

**Metadata** (метаданные) — это "данные о данных". В JDBC они помогают получить информацию о структуре базы данных, таблицах, колонках, типах данных и других свойствах, не зная их заранее.

Представь, что база данных — это книга, а **метаданные** — это оглавление, список глав и страниц. Они не содержат самих данных, но говорят, как эти данные устроены.

---

#### **Какие бывают метаданные в JDBC?**

1. **DatabaseMetaData** — информация о всей базе данных:
    
    - Какие таблицы есть?
        
    - Какие хранимые процедуры?
        
    - Поддерживает ли база транзакции?
        
    - Какая версия СУБД?
        
2. **ResultSetMetaData** — информация о результате запроса (ResultSet):
    
    - Сколько колонок в результате?
        
    - Какие у них названия и типы данных?
        
    - Можно ли изменять эти колонки?


---

### **Примеры использования**

#### **1. DatabaseMetaData (информация о БД)**

```java
Connection connection = DriverManager.getConnection("jdbc:postgresql://localhost:5432/test", "user", "password");

// Получаем метаданные БД
DatabaseMetaData dbMetaData = connection.getMetaData();

// Примеры запросов:
System.out.println("Имя СУБД: " + dbMetaData.getDatabaseProductName()); // PostgreSQL
System.out.println("Версия: " + dbMetaData.getDatabaseProductVersion()); // 15.3
System.out.println("Драйвер: " + dbMetaData.getDriverName()); // PostgreSQL JDBC Driver

// Список всех таблиц в БД
ResultSet tables = dbMetaData.getTables(null, null, "%", new String[]{"TABLE"});
while (tables.next()) {
    System.out.println("Таблица: " + tables.getString("TABLE_NAME"));
}
```

#### **2. ResultSetMetaData (информация о результате запроса)**

```java
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery("SELECT * FROM users");

// Получаем метаданные результата
ResultSetMetaData rsMetaData = resultSet.getMetaData();

// Количество колонок
int columnCount = rsMetaData.getColumnCount();
System.out.println("Колонок в результате: " + columnCount);

// Информация о каждой колонке
for (int i = 1; i <= columnCount; i++) {
    System.out.println("---");
    System.out.println("Имя колонки: " + rsMetaData.getColumnName(i));
    System.out.println("Тип данных: " + rsMetaData.getColumnTypeName(i));
    System.out.println("Максимальная длина: " + rsMetaData.getColumnDisplaySize(i));
}
```


### **Зачем это нужно?**

1. **Динамические запросы**  
    Если пишешь универсальный код, который должен работать с разными таблицами, метаданные помогут узнать структуру таблицы на лету.
    
2. **Генерация отчётов**  
    Можно автоматически создавать CSV или Excel из ResultSet, используя названия колонок из метаданных.
    
3. **Валидация**  
    Проверить, совпадает ли структура таблицы с ожидаемой (например, есть ли нужные колонки).
    
4. **Логирование и отладка**  
    Узнать, какие данные возвращает запрос, без вывода самих строк.
    

---

### **Важные методы**

#### **DatabaseMetaData**

|Метод|Описание|
|---|---|
|`getTables()`|Список всех таблиц|
|`getColumns()`|Колонки таблицы|
|`getPrimaryKeys()`|Первичные ключи таблицы|
|`supportsTransactions()`|Поддерживает ли БД транзакции?|

#### **ResultSetMetaData**

|Метод|Описание|
|---|---|
|`getColumnCount()`|Сколько колонок в ResultSet?|
|`getColumnName(i)`|Имя колонки по индексу|
|`getColumnTypeName(i)`|Тип данных колонки (VARCHAR, INT и т.д.)|
|`isNullable(i)`|Может ли колонка быть NULL?|

---

### **Аналогия**

Представь, что ты заходишь в библиотеку:

- **DatabaseMetaData** — это каталог библиотеки (список всех книг).
    
- **ResultSetMetaData** — это оглавление конкретной книги (названия глав и страниц).
    

Сама книга — это данные (ResultSet), а метаданные помогают понять, где что лежит.

---

### **Итог**

**Metadata в JDBC** — это мощный инструмент для работы с неизвестной структурой базы данных. Они позволяют писать гибкий код, который адаптируется под разные таблицы и БД без жёсткой привязки к схеме.



### Транзакции


**Транзакция** — это группа SQL-запросов, которые выполняются **как единое целое**. Если один запрос в транзакции fails (например, из-за ошибки), то **все изменения откатываются**, как будто ничего не происходило.

`connection.setAutoCommit(false)` - перевести все управление транзакциями на себя

И после того как мы выполним все нужные нам запросы, можно просто ввести:
```java
connection.commit();
```


#### 🔄 **Как это работает?**

1. **Начало транзакции** — говорим JDBC: "Сейчас будет несколько операций, которые надо выполнить вместе".
    
2. **Выполнение запросов** — INSERT, UPDATE, DELETE и т.д.
    
3. **Фиксация (Commit)** — если всё успешно, сохраняем изменения.
    
4. **Откат (Rollback)** — если что-то пошло не так, отменяем все изменения.
    

---

### **Пример: Перевод денег между счетами**

Допустим, мы переводим 100$ со счета `Алисы` на счет `Боба`.  
**Важно:** если списание прошло, а зачисление — нет, деньги не должны пропасть!

```java
Connection connection = null;
try {
    // 1. Получаем соединение с БД
    connection = DriverManager.getConnection("jdbc:postgresql://localhost:5432/bank", "user", "password");
    
    // 2. Отключаем авто-коммит (иначе каждый запрос выполняется отдельно)
    connection.setAutoCommit(false);  // ⚠️ Важно!
    
    // 3. Выполняем запросы в транзакции
    PreparedStatement withdraw = connection.prepareStatement(
        "UPDATE accounts SET balance = balance - 100 WHERE user = 'Алиса'");
    withdraw.executeUpdate();
    
    PreparedStatement deposit = connection.prepareStatement(
        "UPDATE accounts SET balance = balance + 100 WHERE user = 'Боб'");
    deposit.executeUpdate();
    
    // 4. Если всё ок — сохраняем изменения
    connection.commit();  
    System.out.println("Перевод успешен!");
    
} catch (SQLException e) {
    // 5. Если ошибка — откатываем всё
    if (connection != null) {
        connection.rollback();  // Отмена всех изменений
    }
    System.out.println("Ошибка! Перевод отменён.");
} finally {
    // 6. Закрываем соединение
    if (connection != null) {
        connection.setAutoCommit(true);  // Возвращаем авто-коммит
        connection.close();
    }
}
```

### **Основные методы для работы с транзакциями**

|Метод|Что делает|
|---|---|
|`setAutoCommit(false)`|Отключает автосохранение (транзакция не завершится, пока не будет `commit()`).|
|`commit()`|Сохраняет все изменения в БД.|
|`rollback()`|Отменяет все изменения с момента начала транзакции.|
|`setSavepoint()`|Создает точку сохранения внутри транзакции (можно откатить не всю транзакцию, а до этой точки).|

---

### **Зачем это нужно?**

1. **Целостность данных**
    
    - Если ошибка — данные не "ломаются" (например, деньги не исчезают при переводе).
        
2. **Удобство**
    
    - Можно выполнить сложную логику из нескольких запросов, но гарантировать, что либо всё выполнится, либо ничего.
        
3. **Контроль**
    
    - Решаем сами, когда сохранять изменения (`commit`), а когда отменять (`rollback`).
        

---

### **Типичные сценарии**

- **Банковские операции** (переводы, платежи).
    
- **Регистрация пользователя** (создание профиля + настройки + отправка письма — либо всё, либо ничего).
    
- **Импорт данных** (если ошибка в строке 1000 — откатываем весь импорт).
    

---

### **Важные нюансы**

1. **Авто-коммит по умолчанию включен**
    
    - Если не вызвать `setAutoCommit(false)`, каждый запрос будет отдельной транзакцией.
        
2. **Транзакции поддерживают не все БД**
    
    - Например, в MySQL они работают только для таблиц InnoDB (не MyISAM).
        
3. **Уровни изоляции**
    
    - Можно настраивать, как транзакции видят изменения друг друга (например, `READ_COMMITTED`, `SERIALIZABLE`).
        

---

### **Аналогия из жизни**

Представь, что транзакция — это **покупка в интернет-магазине**:

1. Ты добавляешь товары в корзину (`INSERT`).
    
2. Вводишь данные карты (`UPDATE` баланса).
    
3. Если оплата прошла — заказ создается (`COMMIT`).
    
4. Если карта отказала — корзина очищается (`ROLLBACK`).
    

---

### **Итог**

Транзакции в JDBC — это **безопасный способ выполнять несколько запросов как единое целое**. Они защищают данные от "поломки" при ошибках.


### Batch запросы


**Batch (пакет)** — это способ отправить **несколько SQL-запросов в базу данных одним "пакетом"**, вместо того чтобы выполнять их по одному.

Представь, что ты грузишь коробки в фуру:

- **Обычные запросы** — как носить по одной коробке за раз (медленно).
    
- **Batch-запросы** — как загрузить сразу 100 коробок за один рейс (быстро).
    

---

### **Зачем нужны Batch-запросы?**

1. **Ускорение работы**
    
    - Отправка одного большого пакета быстрее, чем 100 маленьких запросов.
        
2. **Снижение нагрузки на сеть**
    
    - Меньше "разговоров" между программой и базой данных.
        
3. **Удобство**
    
    - Можно добавить много запросов в список и выполнить их одной командой.
        

---

### **Как это работает?**

1. Создаёшь SQL-запрос (например, `INSERT`).
    
2. Добавляешь его в "пакет" (batch).
    
3. Повторяешь для других запросов.
    
4. Отправляешь весь пакет в базу одной командой.
    

---

#### **Пример (добавление 3 пользователей в таблицу)**
```java
try (Connection connection = DriverManager.getConnection(url, user, password)) {
    // 1. Создаём Statement или PreparedStatement
    String sql = "INSERT INTO users (name, age) VALUES (?, ?)";
    PreparedStatement statement = connection.prepareStatement(sql);
    
    // 2. Добавляем запросы в пакет
    statement.setString(1, "Алекс");
    statement.setInt(2, 25);
    statement.addBatch();  // Добавили первый запрос
    
    statement.setString(1, "Мария");
    statement.setInt(2, 30);
    statement.addBatch();  // Добавили второй запрос
    
    statement.setString(1, "Иван");
    statement.setInt(2, 28);
    statement.addBatch();  // Добавили третий запрос
    
    // 3. Выполняем весь пакет разом
    int[] results = statement.executeBatch();  // Отправляем!
    
    System.out.println("Добавлено записей: " + results.length);
} catch (SQLException e) {
    e.printStackTrace();
}
```

#### **Что возвращает `executeBatch()`?**

Метод возвращает массив `int[]`, где каждое число — это количество изменённых строк для каждого запроса в пакете.



### **Когда использовать Batch?**

- **Массовые вставки** (`INSERT` 1000 строк).
    
- **Массовые обновления** (`UPDATE`/`DELETE`).
    
- **Импорт данных** из файла в базу.
    

⚠️ **Не для всех случаев!**

- Обычные одиночные запросы (`SELECT`) лучше выполнять без batch.


### **Оптимизация**

Чтобы batch работал ещё быстрее:

1. **Отключи авто-коммит**
```java
connection.setAutoCommit(false);  // Выключаем автосохранение
statement.executeBatch();
connection.commit();  // Сохраняем изменения вручную
```
2. **Используй `PreparedStatement`** (как в примере выше).
    
3. **Разбивай огромные пакеты** на части (например, по 1000 запросов).
    

---

### **Аналог в реальной жизни**

- **Обычный запрос** — как отправить 10 писем по одному (каждый раз идти на почту).
    
- **Batch-запрос** — как отправить 10 писем разом (один поход на почту).
    

---

### **Итог**

Batch-запросы в JDBC — это **способ выполнить много операций за один раз**, что ускоряет работу с базой.






### Blob & Clob


**Blob** - binary large object (в Posgresql - это byte array)
**Clob** - character large object (в Postgresql - это TEXT)




### Пул соединений


**Connection Pool** - это пул соединений, то ест список соединений которые открываются в самом начале программы и потом переиспользуются.


Чтобы создать пул соединений, изменим класс ConnectionManager с помощью которого мы создавали соединения:


```java
public final class ConnectionManager {  
    private static final String URL_KEY = "db.url";  
    private static final String USERNAME_KEY = "db.username";  
    private static final String PASSWORD_KEY = "db.password";  
    private static final String POOL_SIZE_KEY = "db.pool_size";  
    private static final Integer DEFAULT_POOL_SIZE = 10;  
    private static BlockingQueue<Connection> connectionPool;  
    private static List<Connection> sourceConnections;  
  
    static {  
        try {  
            Class.forName("org.postgresql.Driver");  
        } catch (ClassNotFoundException e) {  
            throw new RuntimeException(e);  
        }  
        initConnectionPool();  
    }  
  
    private static void initConnectionPool() {  
        var poolSize = PropertiesUtil.get(POOL_SIZE_KEY);  
        var size = poolSize == null ? DEFAULT_POOL_SIZE : Integer.parseInt(poolSize);  
        connectionPool = new ArrayBlockingQueue<>(size);  
        sourceConnections = new ArrayList<>(size);  
        for (int i = 0; i < size; i++) {  
            var connection = open();  
            var proxyConnection = (Connection)  
                    Proxy.newProxyInstance(ConnectionManager.class.getClassLoader(), new Class[]{Connection.class},  
                    (proxy, method, args) -> method.getName().equals("close") ? connectionPool.add((Connection)proxy)  
                                                                                                        : method.invoke(connection, args));  
            connectionPool.add(proxyConnection);  
            sourceConnections.add(connection);  
        }  
    }  
  
    public static Connection getConnection() {  
        try {  
            return connectionPool.take();  
        } catch (InterruptedException e) {  
            throw new RuntimeException(e);  
        }  
    }  
  
    private ConnectionManager() {}  
  
    private static Connection open() {  
        try {  
            return DriverManager.getConnection(  
                    PropertiesUtil.get(URL_KEY),  
                    PropertiesUtil.get(USERNAME_KEY),  
                    PropertiesUtil.get(PASSWORD_KEY)  
            );  
        } catch (SQLException e) {  
            throw new RuntimeException(e);  
        }  
    }  
    public static void closePool(Connection connection) {  
        for (Connection sourceConnection : sourceConnections) {  
            try {  
                sourceConnection.close();  
            } catch (SQLException e) {  
                throw new RuntimeException(e);  
            }  
        }  
    }  
}
```


#### Что поменялось?

- Добавил в файл .properties поле db.pool_size для того чтобы хранить размер пул соединений
- Создал метод `initConnectionPool` который как раз создает пул соединений:
```java
private static void initConnectionPool() {  
        var poolSize = PropertiesUtil.get(POOL_SIZE_KEY);  
        var size = poolSize == null ? DEFAULT_POOL_SIZE : Integer.parseInt(poolSize);  
        connectionPool = new ArrayBlockingQueue<>(size);  
        sourceConnections = new ArrayList<>(size);  
        for (int i = 0; i < size; i++) {  
            var connection = open();  
            var proxyConnection = (Connection)  
                    Proxy.newProxyInstance(ConnectionManager.class.getClassLoader(), new Class[]{Connection.class},  
                    (proxy, method, args) -> method.getName().equals("close") ? connectionPool.add((Connection)proxy)  
                                                                                                        : method.invoke(connection, args));  
            connectionPool.add(proxyConnection);  
            sourceConnections.add(connection);  
        }  
    }  
  
```

Объяснение работы метода:

```java
var poolSize = PropertiesUtil.get(POOL_SIZE_KEY);  
        var size = poolSize == null ? DEFAULT_POOL_SIZE : Integer.parseInt(poolSize);  
        connectionPool = new ArrayBlockingQueue<>(size);  
        sourceConnections = new ArrayList<>(size);  
```
сначала получаем размер пула из файла `.properties`, потом создаю переменную `size` которая принимает дефолтное значение если в файле не указан размер пула, а если указан то получаем его.
Далее инициализирую блокирующую очередь `connectionPool` и список `sourceConnection`. `sourceConnection` нужен чтобы хранить соединения чтобы потом их закрыть.


```java
for (int i = 0; i < size; i++) {  
            var connection = open();  
            var proxyConnection = (Connection)  
                    Proxy.newProxyInstance(ConnectionManager.class.getClassLoader(), new Class[]{Connection.class},  
                    (proxy, method, args) -> method.getName().equals("close") ? connectionPool.add((Connection)proxy)  
                                                                                                        : method.invoke(connection, args));  
            connectionPool.add(proxyConnection);  
            sourceConnections.add(connection);  
        }  
```

открываю соединения в размере пула соединений при помощи цикла `for`.
```java
var proxyConnection = (Connection)  
                    Proxy.newProxyInstance(ConnectionManager.class.getClassLoader(), new Class[]{Connection.class},  
                    (proxy, method, args) -> method.getName().equals("close") ? connectionPool.add((Connection)proxy)  
                                                                                                        : method.invoke(connection, args));  
```
Этот код создаёт "фальшивое" (прокси) соединение с базой данных, которое ведёт себя почти как настоящее соединение, но с одной хитростью - когда ты вызываешь метод `close()`, вместо реального закрытия соединения, оно возвращается в пул соединений для повторного использования.

##### Разберём по кусочкам:

1. **`Proxy.newProxyInstance`** - это способ создать "фальшивый" объект в Java
    
    - Как будто мы делаем куклу, которая выглядит как настоящее соединение с БД
        
2. **`ConnectionManager.class.getClassLoader()`** - говорит, откуда брать информацию о классах
    
    - Как будто спрашиваем: "Где у нас лежат инструкции для создания этого объекта?"
        
3. **`new Class[]{Connection.class}`** - указывает, какой интерфейс мы имитируем
    
    - Говорим: "Наша кукла должна выглядеть как объект Connection (соединение с БД)"
        
4. **Вот самое интересное - третий аргумент (лямбда-выражение)**:
```java
(proxy, method, args) -> method.getName().equals("close") 
    ? connectionPool.add((Connection)proxy) 
    : method.invoke(connection, args)
```
5. Это "мозги" нашей куклы. Здесь решается:
    
    - Если вызывается метод `close()`:
        
        - То мы не закрываем соединение по-настоящему
            
        - А возвращаем его в пул соединений (`connectionPool.add`)
            
    - Если вызывается ЛЮБОЙ другой метод:
        
        - Мы просто передаём вызов настоящему соединению (`method.invoke(connection, args)`)
            

##### Простая аналогия:

Представь, что у тебя есть секретарь (прокси), который принимает все звонки вместо тебя:

- Если звонят, чтобы просто поговорить - секретарь соединяет с тобой (вызывает настоящий метод)
    
- Если звонят, чтобы попрощаться (вызов `close()`) - секретарь не беспокоит тебя, а просто записывает, что ты снова свободен (возвращает соединение в пул)
    

##### Зачем это нужно?

Чтобы не создавать новое соединение с БД каждый раз, что очень "дорого" по времени. Вместо этого мы:

1. Берём соединение из пула
    
2. Используем
    
3. Когда программа думает, что закрывает его - мы просто возвращаем его в пул
    
4. Теперь его можно использовать снова
    

Это как аренда велосипедов - вместо того чтобы покупать новый каждый раз, ты берёшь из парка, катаешься, а потом возвращаешь обратно для других.




```java
public static Connection getConnection() {  
    try {  
        return connectionPool.take();  
    } catch (InterruptedException e) {  
        throw new RuntimeException(e);  
    }  
}
```
Здесь просто создал метод который берет соединение из пула




```java
public static void closePool(Connection connection) {  
        for (Connection sourceConnection : sourceConnections) {  
            try {  
                sourceConnection.close();  
            } catch (SQLException e) {  
                throw new RuntimeException(e);  
            }  
        }  
    }
```
А здесь создал метод который закрывает соединения. Он берет соединения из `sourceConnection` и вызывает у каждого соединения метод `close`






## DAO



![[Pasted image 20250327195353.png]]



**DAO (Data Access Object)** - нужен для того чтобы преобразовывать реляционную модель в объектную модель и наоборот.


![[Pasted image 20250327195742.png]]


Как используется DAO?
Мы создаем Java Class TicketDao (где Ticket - название таблицы). И в нем создаем методы для работы с БД. 
Далее создаем сущность - это Java Class Ticket, который описывает таблицу Ticket и все ее поля.



![[Pasted image 20250327200246.png]]


Здесь показана слоистая архитектура. Тут суть в том что все остальные сервисы обращаются именно к DAO а не к базе данных и сущностям.






### DAO. Entity mapping


Вот пример создания сущности по таблицы Info (id, data):

```java
public class Info {  
    private Integer id;  
    private String data;  
  
    public Info(Integer id, String data) {  
        this.id = id;  
        this.data = data;  
    }  
  
    public Integer getId() {  
        return id;  
    }  
  
    public void setId(Integer id) {  
        this.id = id;  
    }  
  
    public String getData() {  
        return data;  
    }  
  
    public void setData(String data) {  
        this.data = data;  
    }  
  
    @Override  
    public String toString() {  
        return "Info{" +  
                "id=" + id +  
                ", data='" + data + '\'' +  
                '}';  
    }  
}
```


А вот пример создания класса DAO:

```java
public class InfoDao {  
    private static final InfoDao INSTANCE = new InfoDao();  
    private InfoDao() {}  
      
    public static InfoDao getInstance() {  
        return INSTANCE;  
    }  
}
```

DAO должен реализовать паттерн singleton, то есть должен существовать только один объект данного класса



### DAO. Операции DELETE и INSERT


Вот пример создания метода `delete` который удаляет значение из таблицы по id:

```java
public boolean delete(Integer id) {  
    try (var connection = ConnectionManager.getConnection();  
         PreparedStatement preparedStatement = connection.prepareStatement(DELETE_SQL)) {  
        preparedStatement.setInt(1, id);  
        return preparedStatement.executeUpdate() > 0;  
    } catch (SQLException e) {  
        throw new DaoException(e);  
    }  
}
```

Перед этим еще нужно описать `DaoException`:
```java
public class DaoException extends RuntimeException {  
    public DaoException(Throwable throwable) {  
        super(throwable);  
    }  
}
```

И в классе InfoDao записать константное значение `DELETE_SQL`:
```java
private static final String DELETE_SQL = """  
        DELETE FROM info  
        WHERE id = ?  
        """;
```


 А вот пример реализации метода `save` который вставляет значение в таблицу:
```java
public Info save(Info info) {  
    try (Connection connection = ConnectionManager.getConnection();  
         PreparedStatement preparedStatement = connection.prepareStatement(SAVE_SQL, Statement.RETURN_GENERATED_KEYS)) {  
        preparedStatement.setString(1, info.getData());  
        preparedStatement.executeUpdate();  
  
        var key = preparedStatement.getGeneratedKeys();  
        if (key.next()) {  
            info.setId(key.getInt("id"));  
        }  
        return info;  
    } catch (SQLException e) {  
        throw new DaoException(e);  
    }  
}
```


А вот как можно воспользоваться данными методами:
```java
InfoDao infoDao = InfoDao.getInstance();  
Info info = new Info();  
info.setData("Test Data");  
  
var savedInfo = infoDao.save(info);  
System.out.println(savedInfo);
```
Сначала получил объект DAO, потом создал объект сущности таблицы Info и установил значение. Далее выполнил метод `save` и сохранил его в переменную `savedInfo`.
Вот такой вывод:
```
Info{id=9, data='Test Data'}
```


Ну а вот пример использования метода `delete`:
```java
InfoDao infoDao = InfoDao.getInstance();  
var deleteInfo = infoDao.delete(5);  
System.out.println(deleteInfo);
```





#### DAO. Операции UPDATE и SELECT 



Вот пример метода `update`, который обновляет значения в таблице:
```java
public void update(Info info) {  
    try (Connection connection = ConnectionManager.getConnection();  
         PreparedStatement preparedStatement = connection.prepareStatement(UPDATE_SQL)) {  
        preparedStatement.setString(1, info.getData());  
        preparedStatement.setInt(2, info.getId());  
  
        preparedStatement.executeUpdate();  
    } catch (SQLException e) {  
        throw new DaoException(e);  
    }  
}
```

А вот такой SQL-запрос:
```java
private static final String UPDATE_SQL = """  
        UPDATE info   
		SET data = ?  
        WHERE id = ?  
        """;
```




Ну а вот так выглядит метод `findById` который выполняет `SELECT` запрос на поиск строки по id:
```java
public Optional<Info> findById(int id) {  
    try(Connection connection = ConnectionManager.getConnection();  
        PreparedStatement preparedStatement = connection.prepareStatement(FIND_BY_ID_SQL)) {  
        preparedStatement.setInt(1, id);  
  
        ResultSet resultSet = preparedStatement.executeQuery();  
        Info info = null;  
        if (resultSet.next()) {  
            info = new Info(  
                    resultSet.getInt("id"),  
                    resultSet.getString("data")  
            );  
        }  
        return Optional.ofNullable(info);  
    } catch (SQLException e) {  
        throw new DaoException(e);  
    }  
}
```
`Optional` мы возвращаем потому что значение может быть `NULL` и в таком случае возвращать `Optional` является хорошим тоном.


А вот пример использования метода `findById`:
```java
InfoDao infoDao = InfoDao.getInstance();  
Optional<Info> info = infoDao.findById(6);  
System.out.println(info);
```



А вот совместная работа метода `findById` и `update` - сначала находим объект, а потом обновляем в нем значение
```java
InfoDao infoDao = InfoDao.getInstance();  
Optional<Info> info = infoDao.findById(6);  // нашли объект?
  
  
info.ifPresent(i -> {  // если он существует
    i.setData("New Data");  // установить новое значение
    infoDao.update(i);  // применяем метод update и обновляем данные
});
```



А вот пример метода `findAll` который выполняем `SELECT`-запрос на вывод всех строк из таблицы:
```java
public List<Info> findAll() {  
    try(Connection connection = ConnectionManager.getConnection();  
    PreparedStatement preparedStatement = connection.prepareStatement(FIND_ALL_SQL)) {  
        ResultSet resultSet = preparedStatement.executeQuery();  
        List<Info> list = new ArrayList<>();  
        while(resultSet.next()) {  
            list.add(buildInfo(resultSet));  
        }  
        return list;  
    } catch (SQLException e) {  
        throw new DaoException(e);  
    }  
}
```

Создание объекта `info` я выделил в отдельный метод `buildInfo`:
```java
private static Info buildInfo(ResultSet resultSet) throws SQLException {  
    return new Info(  
            resultSet.getInt("id"),  
            resultSet.getString("data")  
    );  
}
```


Вот пример использования данного метода:
```java
InfoDao infoDao = InfoDao.getInstance();  
List<Info> infos = infoDao.findAll();  
for (Info info : infos) {  
    System.out.println(info);  
}
```
Вывод будет такой:
```
Info{id=1, data='Test1'}
Info{id=2, data='Test2'}
Info{id=3, data='Test3'}
Info{id=4, data='Test4'}
Info{id=7, data='Test7'}
Info{id=8, data='Test8'}
Info{id=9, data='Test Data'}
Info{id=6, data='New Data'}
```




#### DAO. Batch SELECT с фильтрацией



Вот пример метода `findAll(InfoFilter filter)` который делает SELECT с фильрацией добавляя `LIMIT` и `OFFSET`:
```java
public List<Info> findAll(InfoFilter filter) {  
    List<Object> params = new ArrayList<>();  
    params.add(filter.limit());  
    params.add(filter.offset());  
  
    String sql = FIND_ALL_SQL + """  
            LIMIT ?  
            OFFSET ?  
            """;  
    try (Connection connection = ConnectionManager.getConnection();  
    PreparedStatement preparedStatement = connection.prepareStatement(sql)) {  
        for (int i = 0; i < params.size(); i++) {  
            preparedStatement.setObject(i + 1, params.get(i));  
        }  
        ResultSet resultSet = preparedStatement.executeQuery();  
        List<Info> list = new ArrayList<>();  
        while(resultSet.next()) {  
            list.add(buildInfo(resultSet));  
        }  
        return list;  
    } catch (SQLException e) {  
        throw new DaoException(e);  
    }  
}
```
Сначала перед этим в пакете `dto` создаем record `InfoFilter`:
```java
public record InfoFilter (int limit, int offset) {  
}
```
Потом создаем список параметров которые будут добавляться в SQL-запрос:
```java
List<Object> params = new ArrayList<>();  
    params.add(filter.limit());  
    params.add(filter.offset()); 
```
Далее формирует SQL-запрос и устанавливаем соединение. Потом в цикле проходим по параметрам и устанавливаем значение с помощью стэйтмента:
```java
for (int i = 0; i < params.size(); i++) {  
            preparedStatement.setObject(i + 1, params.get(i));  
    }
```
Далее выполняем запрос и добавляем данные в лист и возвращаем его:
```java
ResultSet resultSet = preparedStatement.executeQuery();  
        List<Info> list = new ArrayList<>();  
        while(resultSet.next()) {  
            list.add(buildInfo(resultSet));  
        }  
        return list;  
```




#### DAO. Сложный entity mapping


##### Возьмём пример: Блог с постами и комментариями

У нас есть 2 таблицы:

1. **Посты** (posts)
    
    - id (первичный ключ)
        
    - title (заголовок)
        
    - content (текст поста)
        
2. **Комментарии** (comments)
    
    - id (первичный ключ)
        
    - text (текст комментария)
        
    - post_id (внешний ключ к posts.id)
        

Между ними связь **Один-ко-Многим** (один пост → много комментариев)

---

##### Как это представить в Java-коде?

###### 1. Создаём Java-классы (Entity)

**Post.java** (Пост)
```java
public class Post {
    private int id;
    private String title;
    private String content;
    private List<Comment> comments; // Здесь храним все комментарии к посту
    
    // Конструкторы, геттеры и сеттеры
}
```


**Comment.java** (Комментарий)
```java
public class Comment {
    private int id;
    private String text;
    private int postId; // Ссылка на пост, к которому относится
    
    // Конструкторы, геттеры и сеттеры
}
```



##### 2. Создаём DAO (Data Access Object)

Это "посредники" между базой данных и нашей программой.

##### PostDao.java
```java
public class PostDao {
    // Метод для получения поста со всеми комментариями
    public Post getPostWithComments(int postId) {
        Post post = new Post();
        
        // 1. Получаем сам пост
        String postSql = "SELECT * FROM posts WHERE id = ?";
        try (Connection conn = getConnection();
             PreparedStatement stmt = conn.prepareStatement(postSql)) {
            
            stmt.setInt(1, postId);
            ResultSet rs = stmt.executeQuery();
            
            if (rs.next()) {
                post.setId(rs.getInt("id"));
                post.setTitle(rs.getString("title"));
                post.setContent(rs.getString("content"));
            }
        }
        
        // 2. Получаем все комментарии к этому посту
        String commentsSql = "SELECT * FROM comments WHERE post_id = ?";
        List<Comment> comments = new ArrayList<>();
        
        try (Connection conn = getConnection();
             PreparedStatement stmt = conn.prepareStatement(commentsSql)) {
            
            stmt.setInt(1, postId);
            ResultSet rs = stmt.executeQuery();
            
            while (rs.next()) {
                Comment comment = new Comment();
                comment.setId(rs.getInt("id"));
                comment.setText(rs.getString("text"));
                comment.setPostId(rs.getInt("post_id"));
                
                comments.add(comment);
            }
        }
        
        // 3. Добавляем комментарии к посту
        post.setComments(comments);
        
        return post;
    }
}
```


##### CommentDao.java
```java
public class CommentDao {
    // Метод для добавления комментария к посту
    public void addComment(Comment comment) {
        String sql = "INSERT INTO comments (text, post_id) VALUES (?, ?)";
        
        try (Connection conn = getConnection();
             PreparedStatement stmt = conn.prepareStatement(sql)) {
            
            stmt.setString(1, comment.getText());
            stmt.setInt(2, comment.getPostId());
            stmt.executeUpdate();
        }
    }
}
```


##### Как это работает на практике?

1. **Получаем пост с комментариями:**
```java
PostDao postDao = new PostDao();
Post post = postDao.getPostWithComments(1); // Получаем пост с id=1 и все его комментарии

System.out.println("Пост: " + post.getTitle());
System.out.println("Комментарии:");
for (Comment comment : post.getComments()) {
    System.out.println("- " + comment.getText());
}
```

2. **Добавляем новый комментарий:**
```java
CommentDao commentDao = new CommentDao();

Comment newComment = new Comment();
newComment.setText("Отличный пост!");
newComment.setPostId(1); // Указываем, к какому посту относится

commentDao.addComment(newComment);
```

##### Главные правила при таком маппинге:

1. **Одна сущность = одна таблица** (Post → posts, Comment → comments)
    
2. **Связи между таблицами = поля в объектах** (post_id в Comment → List<>Comment в Post)
    
3. **DAO скрывает сложность работы с БД** - основной код не видит SQL-запросов
    

Это как конструктор Lego:

- Таблицы в БД = отдельные детальки
    
- Наш маппинг = инструкция, как их соединить
    
- Результат = готовый объект со всеми связями
