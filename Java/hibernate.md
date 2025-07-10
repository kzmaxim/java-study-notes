# Hibernate


**ORM (Objec Relational Mapping)** - процесс преобразования объектно-ориентированной модели в реляционную и наоборот

**Hibernate** - инструмент, который автоматизирует процесс преобразования объектно-ориентированной модели в реляционную и наоборот (ORM framework)


Схематично это можно изобразить следующим образом:

[![hibernate](https://i0.wp.com/proselyte.net/wp-content/uploads/2016/02/hibernate-768x260.jpg?resize=700%2C237)](https://i0.wp.com/proselyte.net/wp-content/uploads/2016/02/hibernate.jpg)


![[Pasted image 20250705123751.png]]


![[Pasted image 20250705124014.png]]


Какие же преимущества даёт нам использование Hibernate?

- Обеспечивает простой API для записи и получения Java-объектов в/из БД.
- Минимизирует доступ к БД, используя стратегии fetching.
- Не требует сервера приложения.
- Позволяет нам не работать с типами данных языка SQL, а иметь дело с привычными нам типами данных Java.
- Заботится о создании связей между Java-классами и таблицами БД с помощью XML-файлов не внося изменения в программный код.
- Если нам необходимо изменить БД, то достаточно лишь внести изменения в XML-файлы.


#### Архитектура.

Hibernate – это ORM фреймворк для Java с открытым исходным кодом. Эта технология является крайне мощной и имеет высокие показатели производительности.

Hibernate создаёт связь между таблицами в базе данных (далее – БД) и Java-классами и наоборот. Это избавляет разработчиков от огромного количества лишней, рутинной работы, в которой крайне легко допустить ошибку и крайне трудно потом её найти.

Схематично это можно изобразить следующим образом:

[![hibernate](https://i0.wp.com/proselyte.net/wp-content/uploads/2016/02/hibernate-768x260.jpg?resize=700%2C237)](https://i0.wp.com/proselyte.net/wp-content/uploads/2016/02/hibernate.jpg)

Какие же преимущества даёт нам использование Hibernate?

- Обеспечивает простой API для записи и получения Java-объектов в/из БД.
- Минимизирует доступ к БД, используя стратегии fetching.
- Не требует сервера приложения.
- Позволяет нам не работать с типами данных языка SQL, а иметь дело с привычными нам типами данных Java.
- Заботится о создании связей между Java-классами и таблицами БД с помощью XML-файлов не внося изменения в программный код.
- Если нам необходимо изменить БД, то достаточно лишь внести изменения в XML-файлы.

Hibernate поддерживает все основные СУБД: MySQL, Oracle, PostgreSQL, Microsoft SQL Server Database, HSQL, DB2.  
Hibernate также может работать в связке с такими технологиями, как Maven и J2EE.

**Архитектура**

Приложение, которе использует Hibernate (в крайне поверхностном представлении) имеет такую архитектуру:

[![arch](https://i0.wp.com/proselyte.net/wp-content/uploads/2016/02/arch-768x597.jpg?resize=700%2C544)](https://i0.wp.com/proselyte.net/wp-content/uploads/2016/02/arch.jpg)

Если мы рассмотрим строение самого Hibernate более подробно, что этот же рисунок будет выглядеть следующим образом:

[![hibArchDetailed](https://i0.wp.com/proselyte.net/wp-content/uploads/2016/02/hibArchDetailed-1-768x614.jpg?resize=700%2C560)](https://i0.wp.com/proselyte.net/wp-content/uploads/2016/02/hibArchDetailed-1.jpg)

Давайте рассмотрим отдельно каждый из элементов Hibernate, которые указаны в схеме:

**Transaction**

Этот объект представляет собой рабочую единицу работы с БД. В Hibernate транзакции обрабатываются менеджером транзакций.

**SessionFactory**

Самый важный и самый тяжёлый объект (обычно создаётся в единственном экземпляре, при запуске приложения). Нам необходима как минимум одна SessionFactory для каждой БД, каждый из которых конфигурируется отдельным конфигурационным файлом.

**Session**

Сессия используется для получения физического соединения с БД. Обычно, сессия создаётся при необходимости, а после этого закрывается. Это связано с тем, что эти объекты крайне легковесны. Чтобы понять, что это такое, модно сказать, что создание, чтение, изменение и удаление объектов происходит через объект Session.

![[Pasted image 20250705124128.png]]

![[Pasted image 20250705124228.png]]

![[Pasted image 20250705124505.png]]



**Query**

Этот объект использует HQL или SQL для чтения/записи данных из/в БД. Экземпляр запроса используется для связывания параметров запроса, ограничения количества результатов, которые будут возвращены и для выполнения запроса.

**Configuration**

Этот объект используется для создания объекта SessionFactory и конфигурирует сам Hibernate с помощью конфигурационного XML-файла, который объясняет, как обрабатывать объект Session.

**Criteria**

Используется для создания и выполнения объекто-ориентированного запроса для получения объектов.



## Конфигурация SessionFactory
Дополнительная лекция: https://javarush.com/quests/lectures/questhibernate.level09.lecture02
https://javarush.com/quests/lectures/questhibernate.level09.lecture04



**Session** - это такой аналог Connection 
**SessionFactory** - это аналог Connection Pool

Сначала нужно добавить конфигурационный файл hibernate. Для этого нужно зайти в Modules и добавить Hibernate, а потом в самом модуле Hibernate нажать "+" и добавить конфигурационный файл.
Вот как для начала должен выглядеть этот файл:
```
<?xml version='1.0' encoding='utf-8'?>  
<!DOCTYPE hibernate-configuration PUBLIC  
    "-//Hibernate/Hibernate Configuration DTD//EN"  
    "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">  
<hibernate-configuration>  
  <session-factory>  
    <property name="connection.url">jdbc:postgresql://localhost:5432/university</property>  
    <property name="connection.username">maxim</property>  
    <property name="connection.password">3maxim14</property>  
    <property name="connection.driver_class">org.postgresql.Driver</property>  
    <property name="hibernate.dialect">org.hibernate.dialect.PostgreSQL10Dialect</property>  
  
  
    <!-- DB schema will be updated if needed -->  
    <!-- <property name="hibernate.hbm2ddl.auto">update</property> -->  </session-factory>  
</hibernate-configuration>
```


Также конфигурационный файл для Hibernate можно создать в папке `resources` а именно файл `hibernate.properties`:
```shell
# Конфигурация источника данных (Data source)  
hibernate.driver_class=org.postgresql.Driver  
hibernate.connection.url=jdbc:postgresql://localhost:5432/hibernate_demo_db  
hibernate.connection.username=maxim  
hibernate.connection.password=3maxim14  
  
  
# Конфигурация Hibernate  
hibernate.dialect=org.hibernate.dialect.PostgreSQL10Dialect  
hibernate.show_sql=true  
hibernate.current_session_context_class=thread
```



#### Создание SessionFactory:
```java
Configuration configuration = new Configuration();  
configuration.configure();  
try (SessionFactory sessionFactory = configuration.buildSessionFactory();  
     Session session = sessionFactory.openSession()) {  
    System.out.println("OK");  
}
```

`Configuration` - класс который содержит методы для создания SessionFactory
`configure()` - метод для конфигурации
`buildSessionFactory()` - метод для создания SessionFactory
`openSession()` - создание объекта Session



#### Конфигурирование xml-файла hibernate:

**hibernate.dialect**

Указывает HIebrnate диалект БД. Hibernate, в своб очередь, генерирует необходимые SQL-запросы (например, org.hibernate.dialect.MySQLDialect, если мы используем MySQL).

**hibernate.connection-driver_class**

Указывает класс JDBC драйвера.

**hibernate.connection.url**

Указывает URL (ссылку) необходимой нам БД (например, jdbc:mysql://localhost:3306/database).

**hibernate.connection.username**

Указывает имя пользователя БД (например, root).

**hibernate.connection.password**

Укащывает пароль к БД (например, password).

**hibernate.connection.pool_size**

Ограничивает количество соединений, которые находятся в пуле соединений Hibernate.

**hibernate.connection.autocommit  
**Указывает режим autocommit для JDBC-соединения.






# Entity


Если мы хотим, чтобы экземпляры (объекты) Java-класса в будущем сохранялся в таблице БД, то мы называем их “сохраняемые классы” (persistent class). Для того, чтобы сделать работу с Hibernate максимально удобной и эффективной, мы следует использовать программную модель Простых Старых Java Объектов (Plain Old Java Object – POJO).

Существуют определённые требования к POJO классам. Вот самые главные из них:

- Все классы должны иметь ID для простой идентификации наших объектов в БД и в Hibernate. Это поле класса соединяется с первичным ключом (primary key) таблицы БД.
- Все POJO – классы должны иметь конструктор по умолчанию (пустой).
- Все поля POJO – классов должны иметь модификатор доступа **private** иметь набор getter-ов и setter-ов в стиле JavaBean.
- POJO – классы не должны содержать бизнес-логику.


Для работы с таблицами в Hibernate также нужно создавать сущности как и в JDBC.
Вот пример создания сущности:

```java
@Data  
@NoArgsConstructor  
@AllArgsConstructor  
@Builder  
@Entity  
@Table(name = "test", schema="public")  
public class Test {  
    @Id  
    private Long id;  
    private String reason;  
    private String annotation;  
}
```

`@Data` - это аннотация lombok которая позволяет генерировать геттеры, сеттеры и тд
`@NoArgsConstructor` - генерирует конструктор без аргументов
`@AllArgsConstructor` - генерирует конструктор со всеми параметрами
`@Builder` - генерирует builder для сущности
`@Entity` - аннотация которая помечает что данный класс - это сущности базы данных
`Table(name = "test", schema="public")` - здесь я просто пометил правильное имя таблицы


Чтобы конфигурация отслеживала класс нужно его добавить в файл конфигурации:
```
<mapping class="org.main.entity.Test"/>
```


Работа с сущностью:
```java
Configuration configuration = new Configuration();  
configuration.setPhysicalNamingStrategy(new CamelCaseToUnderscoresNamingStrategy());  
configuration.configure();  
try (SessionFactory sessionFactory = configuration.buildSessionFactory();  
     Session session = sessionFactory.openSession()) {  
    session.beginTransaction();  
    Test builder = Test.builder()  
            .id(1L)  
            .reason("Test reason")  
            .annotation("Test annotation")  
            .build();  
    session.save(builder);  
    session.getTransaction().commit();  
}
```

`configuration.setPhysicalNamingStrategy(new CamelCaseToUnderscoresNamingStrategy()); ` - делает так чтобы все поля в сущности преобразовывались в camel_case. Либо нужно над колонкой написать:
`@Column(name="birth_date")` к примеру

`session.beginTransaction();` - начало транзакции
```java
Test builder = Test.builder()  
            .id(1L)  
            .reason("Test reason")  
            .annotation("Test annotation")  
            .build();
```
создание объекта сущности

`session.save(builder);` - сохранение объекта в базу данных. Выполняется следующая команда:
```sql
insert 
    into
        public.test
        (annotation, reason, id) 
    values
        (?, ?, ?)
```

`session.getTransaction().commit();` - завершение транзакции


# Получение объектов
Осуществляется при помощи такой конструкции:
```java
Класс имя = session.get(Класс.class, ID);
```

Вот пример метода который позволяет получить объект по id:
```java
public User getUserById(Integer id) { 
	try (Session session = sessionFactory.openSession()) { 
		User user = session.get(User.class, id); 
		return user; 
	}
}
```


Вот еще пример получения сущности:
```java
Configuration configuration = new Configuration().addAnnotatedClass( Person.class );  
try (SessionFactory sessionFactory = configuration.buildSessionFactory();  
Session session = sessionFactory.openSession()) {  
  
    session.beginTransaction();  
  
    Person person = session.get(Person.class, 1);  
    System.out.println(person.getName());  
    System.out.println(person.getAge());  
  
    session.getTransaction().commit();  
}
```

# Сохранение (добавление) объектов

Если ты хочешь сохранить свой объект в базу данных, то на уровне SQL будет выполнен запрос ==INSERT==. Поэтому твои действия нужно выполнять в виде отдельной транзакции. Кроме того, для сохранения лучше использовать метод `persist()` объекта **session**.

Общий вид такого запроса:
```java
session.persist(Объект);
```

Метод `persist()` меняет не только базу, но и сам объект. Все дело в том, что когда мы добавляем объект в базу, то до добавления у этого объекта еще нет своего **ID**. Ну, обычно так, хотя бывают нюансы. А после добавления у объекта **ID** уж есть.

```java
public boolean saveUser(User user) {
    try (Session session = sessionFactory.openSession()) {
            Transaction transaction = session.beginTransaction();
            session.persist(user);
            transaction.commit();
            return true;
    }
    catch() {
    return false;
   	}
}
```

Также у объекта **Session** есть метод `save()`, который выполняет аналогичную функцию. Просто метод `save()` — это старый стандарт Hibernate, а метод `persist()` — это JPA-стандарт.


Вот пример с методом `save()`:
```java
Configuration configuration = new Configuration().addAnnotatedClass( Person.class );  
try (SessionFactory sessionFactory = configuration.buildSessionFactory();  
Session session = sessionFactory.openSession()) {  
  
    session.beginTransaction();  
  
    Person person1 = new Person("Vasya", 20);  
    Person person2 = new Person("Sasha", 21);  
    Person person3 = new Person("Glenn", 22);  
  
    session.save( person1 );  
    session.save( person2 );  
    session.save( person3 );  
  
    session.getTransaction().commit();  
}
```


Отличие к том что `save()` возвращает идентификатор но не сохраняет объект в persistence context, а метод `persist()` ничего не возвращает но сохраняет в persistence context

В целом более предпочтительно использовать `persist()`



# Обновление и удаление объектов


## Обновление


Чтобы обновить значение в БД достаточно получить этот объект и вызвать соответствующий сеттер:
```java
try (SessionFactory sessionFactory = configuration.buildSessionFactory();  
Session session = sessionFactory.openSession()) {  
  
    session.beginTransaction();  
  
    Person person = session.get(Person.class, 2);  
    person.setName("Alexandra");  // hibernate сам отправить update запрос
  
    session.getTransaction().commit();  
}
```


## Удаление

Если вы хотите удалить существующей объект, то сделать это очень просто. Для этого у объекта session есть специальный метод — `remove()`.

Общий вид такого запроса:
```java
session.remove(Объект);
```

Пример метода:
```java
public boolean removeUser(User user) {
    try (Session session = sessionFactory.openSession()) {
            Transaction transaction = session.beginTransaction();
            session.remove(user);
            transaction.commit();
            return true;
    }
    catch() {
    return false;
   	}
}
```


Также удалить можно с помощью метода `delete`:
```java
try (SessionFactory sessionFactory = configuration.buildSessionFactory();  
    Session session = sessionFactory.openSession()) {  
  
        session.beginTransaction();  
  
        Person person = session.get(Person.class, 2);  
        session.delete(person);  
  
        session.getTransaction().commit();  
    }  
}
```


Разницы между ними нет. Просто `remove()` это стандарт JPA, а `delete()` - это метод Hibernate



# Язык HQL


![[Pasted image 20250705174445.png]]


![[Pasted image 20250705174533.png]]



![[Pasted image 20250705174658.png]]



Пример с самым простым HQL-запросом: 
```java
List<Person> people = session.createQuery("from Person").getResultList();  
  
for (Person person : people) {  
    System.out.println(person);  
}
```


Вот пример с условием:
```java
List<Person> people = session.createQuery("from Person where age > 20").getResultList();
```


А вот с регулярным выражением:
```java
List<Person> people = session.createQuery("from Person where name like 'G%'").getResultList();
```






# Type Converters


Hibernate сам умеет преобразовывать типы данных. Но только те которые определены в нем. Если нужно преобразовывать кастомный тип данных то нужно реализовать интерфейс `UserType`

Когда ты сохраняешь объекты Java (например, `User`, `Order`, `Product`) в базу данных, Hibernate **автоматически** переводит Java-объекты в SQL-формат (и обратно).

Но бывает, что **Hibernate не знает**, как сохранить какой-то нестандартный тип данных (например, `enum`, `LocalDate`, `List<String>` и т.д.).

Тут на помощь приходят **Type Converters** — это способ **сказать Hibernate, как именно сохранять твой тип данных в базу и обратно.**



Пример с Enum:
```java
public enum Role {
    USER,
    ADMIN
}
```
И использование в сущности:
```java
@Entity
public class User {

    @Id
    private Long id;

    @Enumerated(EnumType.STRING)
    private Role role;

    // геттеры и сеттеры
}
```

#### Что делает `@Enumerated(EnumType.STRING)`?

👉 Он говорит Hibernate: "Сохраняй enum как **строку** в базе данных".

- То есть в базе будет храниться `USER` или `ADMIN`, а не просто число.

#### Type
`@Type` используется, когда ты хочешь **сделать кастомный способ хранения типа**, который Hibernate сам не умеет.

Например:
```java
@Type(type = "json")
private Map<String, Object> settings;
```
Это означает, что ты используешь **кастомный тип**, который сам пишешь или подключаешь.  
Например, чтобы сохранять Map в JSON в базе данных PostgreSQL.


#### 🔨 А если мне надо самому написать свой конвертер?

Ты можешь создать класс, который реализует `AttributeConverter`:
```java
@Converter
public class RoleConverter implements AttributeConverter<Role, String> {

    @Override
    public String convertToDatabaseColumn(Role role) {
        return role.name();
    }

    @Override
    public Role convertToEntityAttribute(String dbData) {
        return Role.valueOf(dbData);
    }
}
```
`Role` - это Java-тип, который нужно преобразовать в SQL-тип
`String` - это SQL-тип в который нужно преобразовать

Либо вот более сложный вариант:
![[Pasted image 20250405205404.png]]


А потом использовать так:
```java
@Convert(converter = RoleConverter.class)
private Role role;
```


Чтобы нам не пришлось каждый раз писать `@Convert` можно добавить в конфигурацию класс который мы конвертируем:
```java
configuration.addAttributeConverter(new BirthDayConverter(), true);
```

Либо над самим классом прописать:
```
@Converter(autoApply=true)
```



Но иногда бывают ситуации когда `AttributeConverter` нам не сможет помочь. Например, когда в SQL нет того типа в который мы хотим преобразовать (например JSON). Чтобы решить эту проблему нужно реализовать класс `Type` или лучше `UserType`. Но там все таки много методов которые нужно переопределять. 
Но на помощь приходит библиотека `Hibernate Types`

После установки библиотеки над полем нужно написать аннотацию `Type`:
`@Type(type = "путь/к/библиотеке")`

Если нужно преобразовать в jSON то используем класс `JsonStringType` или `JsonBinaryType`

А после этого нужно зарегистрировать тип в configuration:
```java
configuration.registerTypeOverride(new JsonBinaryType());
```




## Методы update, delete, get


`update(user)` - делает UPDATE запрос, если переданного объекта не существует то выбрасывает Exception
`saveOrUpdate()` - такой же как `update()`, но сохраняет объект в базе если его не существует
`delete()` - удаляет объект, если объекта нет то не будет выбрасывать исключение
`get(name|type, id)` - делает SELECT запрос по id:
```java
session.get(User.type, 3L);
```




## Entity Persister


![[Pasted image 20250407174027.png]]


**Entity Persister**. Это компонент Hibernate, который управляет тем, как сущности (Java-классы, аннотированные как `@Entity`) взаимодействуют с базой данных. Давай разберемся, что это такое и как работает.

### 1. Что такое **Entity** в Hibernate?

**Entity** — это класс в Java, который представляет таблицу в базе данных. Каждый объект этого класса будет соответствовать одной строке в таблице. В Hibernate, чтобы сделать класс сущностью, его необходимо аннотировать с помощью аннотации `@Entity`. Например:
```java
@Entity
public class Test {
    @Id
    private Long id;
    private String reason;
    private String annotation;

    // Геттеры и сеттеры
}
```
В этом примере класс `Test` — это сущность, которая будет маппиться на таблицу в базе данных.

### 2. Что такое **Entity Persister**?

**Entity Persister** — это часть механизма Hibernate, которая отвечает за процесс **постоянства** объектов сущностей в базе данных. Процесс "постоянства" означает сохранение данных в базе и извлечение этих данных обратно в виде объектов Java. Hibernate использует `Entity Persister`, чтобы преобразовывать объекты Java в строки таблицы базы данных и наоборот.

Каждый раз, когда ты сохраняешь или извлекаешь сущности, **Entity Persister** выполняет работу по маппингу этих данных.

### 3. Как работает **Entity Persister**?

Когда ты вызываешь метод для сохранения, загрузки или обновления сущности, Hibernate обращается к `Entity Persister`, чтобы выполнить необходимые операции с базой данных. Рассмотрим, как это происходит:

#### 1. **Сохранение сущности**:

Когда ты сохраняешь объект с помощью метода `session.save()` или `session.persist()`, Hibernate использует `Entity Persister` для преобразования данных из объекта в SQL-запрос для вставки в таблицу.

Пример:
```java
Session session = sessionFactory.openSession();
session.beginTransaction();

Test test = new Test();
test.setId(1L);
test.setReason("Some reason");
test.setAnnotation("Some annotation");

session.save(test);
session.getTransaction().commit();
session.close();
```

- Hibernate использует `Entity Persister`, чтобы преобразовать объект `test` в SQL-запрос для вставки данных в таблицу `test` базы данных.

#### 2. **Загрузка сущности**:

Когда ты загружаешь объект с помощью метода `session.get()` или `session.load()`, Hibernate использует `Entity Persister` для извлечения данных из таблицы и преобразования их обратно в объект Java.

Пример:
```java
Session session = sessionFactory.openSession();
Test test = session.get(Test.class, 1L);
session.close();
```

- Hibernate использует `Entity Persister`, чтобы выполнить SQL-запрос (например, `SELECT * FROM test WHERE id = 1`) и преобразовать полученные данные в объект `Test`.
    

#### 3. **Обновление сущности**:

Когда ты обновляешь объект с помощью метода `session.update()` или через изменение объекта, Hibernate снова использует `Entity Persister`, чтобы сформировать SQL-запрос для обновления данных в базе данных.

#### 4. **Удаление сущности**:

Когда ты удаляешь объект с помощью `session.delete()`, Hibernate использует `Entity Persister` для создания SQL-запроса для удаления строки из таблицы.

### 4. Как Hibernate управляет сущностями через **Entity Persister**?

Hibernate использует метаинформацию о сущности для того, чтобы правильно взаимодействовать с базой данных. Эта информация включает:

- Имя таблицы, на которую маппится сущность.
    
- Столбцы таблицы, соответствующие полям класса.
    
- Идентификатор (`@Id`), который является первичным ключом.
    
- Типы данных полей и другие параметры.
    

Когда ты работаешь с объектами сущностей, Hibernate использует `Entity Persister` для понимания, как взаимодействовать с базой данных в контексте этих сущностей. Для каждой сущности Hibernate создает соответствующий `Entity Persister`, который знает, как преобразовать объект в SQL-запросы и обратно.

### 5. Пример работы с **Entity Persister**

Предположим, у тебя есть следующая сущность `Test`:
```java
@Entity
@Table(name = "test")
public class Test {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String reason;
    private String annotation;
    // Геттеры и сеттеры
}
```

Теперь, когда ты сохраняешь или загружаешь объект `Test` с помощью Hibernate, например:
```java
Session session = sessionFactory.openSession();
Test test = session.get(Test.class, 1L);
```

Hibernate использует `Entity Persister` для:

1. Выполнения SQL-запроса `SELECT * FROM test WHERE id = 1`.
    
2. Преобразования строки данных из базы данных в объект `Test`.
    

Это происходит с помощью метаинформации, хранящейся в классе `Test`. `Entity Persister` использует эту информацию, чтобы понимать, какие столбцы таблицы соответствуют полям класса.

### Заключение

**Entity Persister** — это важный компонент Hibernate, который управляет процессом сохранения и извлечения данных для сущностей. Он берет на себя работу по маппингу данных между объектами Java и таблицами базы данных, облегчая работу с базой данных в приложениях на Hibernate.

Когда ты начинаешь работать с Hibernate, важно понимать, как работает `Entity Persister`, поскольку это поможет тебе лучше понять внутренние механизмы ORM (объектно-реляционного отображения) и управлять данными в базе данных.



## First Level Cache


`evict(user)` - метод у Session который удаляет сущность из persistance context
`session.clear()` - очистить весь кэш

У каждой сессии есть свой отдельный кэш
Чтобы сущность попала в persistence context (кэш) нужно сделать запрос в базу данных




## Entity Lifecycle

Лекция: https://javarush.com/quests/lectures/questhibernate.level11.lecture00


![[Pasted image 20250407180352.png]]




Можно создать утильный класс для создания сессии:
```java
@UtilityClass  
public class HibernateUtil {  
    public static SessionFactory getSessionFactory() {  
        Configuration configuration = new Configuration();  
        configuration.setPhysicalNamingStrategy(new CamelCaseToUnderscoresNamingStrategy());  
        configuration.configure();  
        return configuration.buildSessionFactory();  
    }  
}
```


`refresh(user)` - метод у session, который синхронизирует объект в java c объектом в базе данных


![](https://cdn.javarush.com/images/article/d79ec4c4-75c1-4ae4-b7c9-8672e38dee58/1024.webp)


### Transient


![[Pasted image 20250706182721.png]]




Это просто POJO объект в Java. Изменения на нем никак не влияют на базу данных.

Если некий клиентский код работает с объектом со статусом Transient, то их взаимодействие можно описать супер-простой схемой:

![](https://cdn.javarush.com/images/article/53224fad-6579-4469-845a-0f1bb13e9a86/256.webp)


### Persistent


![[Pasted image 20250706182833.png]]


![[Pasted image 20250706183037.png]]






Следующий самый распространенный случай – это объекты, связанные с движком Hibernate. Их статус называют Persistent (или же Managed). Способов получить объект с таким статусом ровно два:

- Загрузить объект из Hibernate.
- Сохранить объект в Hibernate.


Взаимодействие клиентского кода и объекта в статусе Managed можно описать вот такой картинкой:

![](https://cdn.javarush.com/images/article/a3dbbcc1-397b-4695-954b-5c52073850ba/800.webp)

### Detached



![[Pasted image 20250706183128.png]]





Следующее состояние – это когда объект был отсоединен от сессии. То есть когда-то объект был присоединен к сессии Hibernate, но затем сессия закрылась или транзакция завершилась, и Hibernate больше не следит за этим объектом.

Новая схема взаимодействия кода и объекта будет выглядеть так:

![](https://cdn.javarush.com/images/article/cd4dfefa-63c3-4026-b48a-b7b2cf5b95e0/800.webp)



### Removed


![[Pasted image 20250706183319.png]]










## Java Persistence API (JPA)


![[Pasted image 20250407182123.png]]






## Logging


![[Pasted image 20250407235620.png]]



Создание логгера:
```java
private static final Logger logger = LoggerFactory.getLogger(Main.class);
```

И пример логгирования:
```java
logger.info("..{}..", arg);
```


Вот пример файла log4f.xml:
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd">  
  
<log4j:configuration>  
  
    <!-- Консольный аппендер -->  
    <appender name="console" class="org.apache.log4j.ConsoleAppender">  
        <layout class="org.apache.log4j.PatternLayout">  
            <param name="ConversionPattern" value="%d{ISO8601} [%t] %-5p %c - %m%n"/>  
        </layout>  
    </appender>  
  
    <!-- Файловый аппендер -->  
    <appender name="fileAppender" class="org.apache.log4j.RollingFileAppender">  
        <param name="File" value="logs/app.log"/>  
        <param name="MaxFileSize" value="10MB"/>  
        <param name="MaxBackupIndex" value="5"/>  
        <layout class="org.apache.log4j.PatternLayout">  
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n"/>  
        </layout>  
    </appender>  
  
    <!-- Уровень логирования по умолчанию -->  
    <root>  
        <priority value="debug"/>  
        <appender-ref ref="console"/>  
        <appender-ref ref="fileAppender"/>  
    </root>  
  
    <!-- Пример: для логгера Hibernate -->  
    <logger name="org.hibernate" additivity="false">  
        <level value="warn"/>  
    </logger>  
    <logger name="org.main" additivity="false">  
        <level value="info"/>  
    </logger>  
  
</log4j:configuration>
```





## Embedded Components


**Embedded component** (встраиваемый компонент) — это **неотдельная сущность**, которую ты **встраиваешь в другую сущность** как часть её полей.

Она:

- **не имеет собственного ID** (не @Entity),
    
- **сохраняется в той же таблице**, что и основная сущность,
    
- просто как **группа полей**, объединённая в один объект.


### 🧠 Зачем это нужно?

Когда у тебя есть **несколько связанных полей**, которые часто используются вместе — удобно объединить их в **один класс**. Это улучшает:

- **структуру кода** (удобнее читать и поддерживать),
    
- **переиспользование** (можно вставлять один и тот же компонент в разные сущности),
    
- **понятность модели** (логически объединённые данные).


Пример: создание emdedded компонента для сущности Test для полей reason и annotation:
```java
@Data  
@NoArgsConstructor  
@AllArgsConstructor  
@Builder  
@Embeddable  
public class TestInfo {  
    private String reason;  
    private String annotation;  
}
```

Сам класс тест изменился так:
```java
@Data  
@NoArgsConstructor  
@AllArgsConstructor  
@Builder  
@Entity  
@Table(name = "test", schema="public")  
public class Test {  
    @Id  
    private Long id;  
    @Embedded  
    private TestInfo testInfo;  // embedded component
}
```

Пример создания объекта:
```java
Test builder = Test.builder()  
        .id(38320L)  
        .testInfo(TestInfo.builder()  // embedded component
                .reason("Test reason 2")  
                .annotation("Test annotation 2")  
                .build())  
        .build();
```




## Primary Keys


Корректное указание для Primary Key:
```java
@Id  
@GeneratedValue(strategy = GenerationType.IDENTITY)  
private Long id;
```

И после этого не нужно указывать id при вставке значения

`GenerationType.IDENTITY` - для полей SERIAL
`GenerationType.SEQUENCE` - используется для sequence если мы их создаем. Нужно использовать @SequenceGenerator
`GenerationType.TABLE` - для использования нужно создания таблицу с sequences где будет хранится имя таблицы как PRIMARY KEY и id этой таблицы. Нужно использовать @TableGenerator








# Mapping entity assiciations

## Mapping даты и времени
Дополнительная лекция: https://javarush.com/quests/lectures/questhibernate.level12.lecture03
и 
https://javarush.com/quests/lectures/questhibernate.level12.lecture04


В нынешнее время с маппингом все обстоит намного проще и лучше. Все базы данных поддерживают 4 типа данных для работы со временем:

- **DATE** – дата: год, месяц и день.
- **TIME** – время: часы, минуты, секунды.
- **TIMESTAMP** – дата, время и наносекунды.
- **TIMESTAMP WITH TIME ZONE** – TIMESTAMP и временная зона (имя зоны или смещение).

Для того, чтобы представить **тип** ==DATE== в Java, нужно использовать класс `java.time.LocalDate` из JDK 8 DateTime API.

Тип ==TIME== из базы данных можно представить двумя типами из Java: `java.time.LocalTime` и `java.time.OffsetTime`. Ничего сложного.

А точную дату и время, представленную типом ==TIMESTAMP== в базе, в Java можно представить 4 типами:

- **java.time.Instant**
- **java.time.LocalDateTime**
- **java.time.OffsetDateTime**
- **java.time.ZonedDateTime**

Ну и наконец ==TIMESTAMP WITH TIME ZONE== можно представить двумя типами:

- **java.time.OffsetDateTime**
- **java.time.ZonedDateTime**


## Mapping коллекций

Для этого используется аннотация `@EntityCollection`

Все поля Entity-класса, которые содержат много элементов и помечаются с помощью аннотации @ElementCollection содержатся в базе данных в специальной вспомогательной таблице. Что, собственно говоря, логично.


Эта таблица может содержать данные в двух видах:

- **Упорядоченные** (List, Map) содержат три колонки:
    - **Key Column** (Foreign Key) – ссылка на ID объекта-родителя.
    - **Index Column** – позиция/индекс в коллекции.
    - **Element Column** – значение.
- **Неупорядоченные** (Set) содержат две колонки:
    - **Key Column** (Foreign Key) – ссылка на ID объекта-родителя.
    - **Element Column** – значение.

Также ты можешь задать имя этой таблицы явно с помощью аннотации:

```java
@CollectionTable(name="имя_таблицы")
```

Пример:
```java
@Entity
@Table(name="user")
class User {
   @Id
   @Column(name="id")
   public Integer id;

   @ElementCollection
   @CollectionTable(name="user_message")
   public List<String> messages;
}
```

**Важно!** Если аннотация @CollectionTable не указана, то Hibernate сам построит имя таблицы на основе имени класса и имени поля: класс User и поле ==messages== дадут имя таблице "User_messages".

## Many to one


Пример:
```java
@ManyToOne  
@JoinColumn(name="contract_type_id")  
private ContractTypes contractTypeId;
```
`@ManyToOne` - показывает связь many to one
`@JoinColumn(name="contract_type_id")` - показывает какая колонка в нашей таблице содержит хранится id объекта ContractType
`private ContractTypes contractTypeId;` - здесь указывается тип с которой идет связь



## Fetch Types

https://javarush.com/quests/lectures/questhibernate.level14.lecture01


#### Представь:

Ты попросил Hibernate загрузить объект `User` с id = 1. Но этот объект может быть большим — у него могут быть вложенные списки, связи с другими таблицами и т.д. Чтобы не загружать всё сразу, Hibernate может "обмануть" тебя:

Он возвращает **не настоящий объект `User`**, а **прокси** — "обёртку", которая выглядит как `User`, но на самом деле данные ещё не загружены. Прокси загружает их **только когда ты к ним обратился**.

### Это называется **ленивая загрузка (Lazy loading)**, и Hibernate делает это через **прокси**.

---

#### 🧩 Пример:
```java
User user = session.get(User.class, 1);
System.out.println(user.getName()); // здесь Hibernate может подгрузить данные, если это прокси
```
Если `user` — это прокси, то когда ты вызываешь `getName()`, Hibernate в этот момент делает запрос к базе и загружает данные.



#### 🧠 Как это работает технически?

Hibernate создаёт **дочерний класс** от твоего класса `User` и внедряет туда специальную логику. Например:
```java
class UserProxy extends User {
    // при вызове getName() Hibernate делает SQL-запрос
}
```



#### ⚠️ На что новичкам стоит обратить внимание

1. **Проблемы вне сессии:** Если ты получишь прокси-объект, но сессия уже закрыта — при попытке обратиться к данным будет ошибка:
```java
LazyInitializationException: could not initialize proxy - no Session
```
2. **Проверка на instanceof:** Прокси — это не совсем `User`, а `User$HibernateProxy`. Так что сравнение `instanceof` может вести себя неожиданно.
    
3.  **Можно отключить прокси:** Можно указать `@Proxy(lazy = false)` или в `@OneToMany(fetch = FetchType.EAGER)` — тогда Hibernate будет загружать данные сразу и не использовать прокси (но это может быть медленно!).


#### 🧪 Вывод

Hibernate Proxy — это хитрый способ сэкономить ресурсы, откладывая загрузку данных до тех пор, пока они реально не понадобятся. Это помогает оптимизировать работу с БД, но требует аккуратности.

Хочешь, я покажу короткий пример с Lazy/Eager загрузкой и прокси в коде?




# Cascade Types (Каскадирование)


`@ManyToOne(cascade = CascadeType.DETACH)` - так мы указываем если мы удаляем сущность то и связанная сущность также удаляется

`@ManyToOne(cascade = CascadeType.PERSIST)` - если сохраняю сущность то и связанная сущность также сохраняется

**Cascade (каскад)** — это когда ты говоришь Hibernate:

> "Эй, если я сделаю что-то с одним объектом, пожалуйста, сделай то же самое и с его связанными объектами!"

---

#### 🔧 Представь ситуацию:

У тебя есть два класса
```java
@Entity
class User {
    @OneToOne(cascade = ???)
    private Profile profile;
}

@Entity
class Profile {
    // ...
}
```

У `User` есть `Profile`. Если ты сохранишь пользователя, хочешь ли ты **автоматически сохранить и его профиль**? Если да — тогда тебе нужен **cascade**.

---

#### 🧩 Какие бывают Cascade Types?

Вот основные типы каскадов:

|Тип|Что делает|
|---|---|
|`PERSIST`|При сохранении (`save`) главного объекта — сохраняется и связанный.|
|`MERGE`|При обновлении (`update`) главного объекта — обновляется и связанный.|
|`REMOVE`|При удалении (`delete`) главного — удаляется и связанный.|
|`REFRESH`|При обновлении из базы (`refresh`) — обновляется и связанный.|
|`DETACH`|При отсоединении от сессии (`detach`) — отсоединяется и связанный.|
|`ALL`|Всё вышеперечисленное.|

---

#### ✅ Пример в коде
```java
@OneToOne(cascade = CascadeType.ALL)
private Profile profile;
```
Теперь, если ты сделаешь:
```java
User user = new User();
Profile profile = new Profile();

user.setProfile(profile);
session.persist(user); // Hibernate сохранит и profile тоже!
```
Без каскада тебе пришлось бы вручную вызывать:
```java
session.persist(profile);
session.persist(user);
```


#### ⚠️ Осторожно с REMOVE

Если ты добавишь `CascadeType.REMOVE`, то при удалении `User` удалится и `Profile`. Это удобно, но **опасно**, если связанные данные нужны другим объектам. Всегда думай: "а не поломаю ли я базу?"

---

#### 📌 Где можно использовать?

- `@OneToOne`
    
- `@OneToMany`
    
- `@ManyToOne`
    
- `@ManyToMany`
    

---

#### 🎓 Запомни просто

**Cascade — это как "эффект домино":** ты делаешь действие над одним объектом, и оно автоматически распространяется на другие, если ты указал нужный тип каскада.





### Еще один пример: мы создали человека и сущность и хотим чтобы при сохранении человека также в базу сохранялись и связанные с ним сущности

Для начала нужно установить каскадирование в классе Person:
```java
@OneToMany(mappedBy = "owner", cascade=CascadeType.PERSIST)  
private List<Item> items;
```

И теперь можно спокойно сохранять человека с помощью метода `persist()` зная, что также сохранятся сущности которые связаны с ним:
```java
session.beginTransaction();  
  
Person person = new Person("Some name", 30);  
  
Item item = new Item("Some item", person);  
  
person.setItems(new ArrayList<>(Collections.singletonList(item)));  
  
session.persist(person);  
  
session.getTransaction().commit();
```



И Hibernate в итоге делает 2 запроса в базу, хоть мы и указали что сохраняем только person:
```sql
Hibernate: insert into person (age, name) values (?, ?)
Hibernate: insert into item (item_name, person_id) values (?, ?)
```



![[Pasted image 20250706184728.png]]


Но пока в моем коде каскадирование работает только с помощью метода `perstist()`. Чтобы работало также с методом `save()` нужно поставить аннотацию `@Cascade()` в класс Person:
```java
@OneToMany(mappedBy = "owner")  
@Cascade(org.hibernate.annotations.CascadeType.SAVE_UPDATE)  
private List<Item> items;
```



Чтобы каскадирование работало также на какие-либо другие методы, можно в фигурных скобках в аннотации `@Cascade` указать эти методы



### Пример с созданием одного человека и нескольких связанных с ним товаров.

Сначала создадим метод в Person который добавляем товар в список и назначает товару владельца:
```java
public void addItem(Item item) {  
    if (this.items == null) {  
        this.items = new ArrayList<>();  
    }  
    this.items.add(item);  
    item.setOwner(this);  
}
```


И далее уже создаем человека и товары и сохраняем все в БД:
```java
session.beginTransaction();  
  
Person person = new Person("Person test", 33);  
  
person.addItem(new Item("Item 1"));  
person.addItem(new Item("Item 2"));  
person.addItem(new Item("Item 3"));  
  
session.save(person);  
  
session.getTransaction().commit();
```




# OneToMany
https://javarush.com/quests/lectures/questhibernate.level13.lecture02


![[Pasted image 20250705185629.png]]








```java
@OneToMany(mappedBy = "company")
private List<User> users;
```
Пример OneToMany связи


А в классе User будет такой код:
```java
@Entity
public class User {
    @ManyToOne
    @JoinColumn(name = "company_id")
    private Company company;
}
```


### 🧠 Теперь по частям:

#### 🧩 `@OneToMany`

Hibernate'у говорим: "Это **один ко многим** — одна компания, много пользователей".

---

#### 🧩 `mappedBy = "company"`

Это **ключевая часть**!

- `mappedBy = "company"` говорит Hibernate'у:
    
    > "Не создавай отдельную таблицу для связи. Связь уже описана в классе `User` — там есть поле `company`."
    
- То есть: **владелец связи — `User`**, а `Company` просто "зеркалит" эту связь.
    

---

### 📊 Как это выглядит в БД?

- Таблица `user` будет содержать колонку `company_id`, которая ссылается на таблицу `company`.
    
- Таблица `company` **не будет иметь списка пользователей** напрямую — это логика реализуется через связи в Java-коде.


### Еще пример:

Есть классы Person и Item. У них отношение OneToMany, то есть один человек может иметь множество предметов, и множество предметов могут иметь одного человека.

Сначала пропишем класс Item:
```java
@Data  
@NoArgsConstructor  
@Entity  
@Table(name="item")  
public class Item {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name="id")  
    private int id;  
  
    @Column(name="item_name")  
    private String itemName;  
  
    @ManyToOne  
    @JoinColumn(name="person_id", referencedColumnName = "id")  
    private Person owner;  
  
  
    public Item(String itemName) {  
        this.itemName = itemName;  
    }  
}
```

Поле `owner` показывает владельца этой вещи. Над этим полем я поставил аннотацию `@ManyToOne` которая показывает отношение. Далее поставил аннотацию `@JoinColumn`
В этой аннотации поля `name` - это имя этого поля в таблице, а `referencedColumnName` это название поля в таблице Person на которую ссылается это поле.


Теперь пропишем класс Person:
```java
@Data  
@NoArgsConstructor  
@Entity  
@Table(name="person")  
public class Person {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name="id")  
    private int id;  
  
    @Column(name="name")  
    private String name;  
  
    @Column(name="age")  
    private int age;  
  
    @OneToMany(mappedBy = "owner")  
    private List<Item> items;  
  
    public Person(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  
  
    @Override  
    public String toString() {  
        return this.name + " " + this.age;  
    }  
}
```


Здесь есть поле `items` которая имеет тип листа. Над этим полем я повесил аннотацию `@OneToMany`. Поле `mappedBy` в этой аннотации означает поле в таблицу Item которая связана с этой таблицей.


## Добавление новых строк при связи OneToMany


```java
session.beginTransaction();  
  
Person person = session.get(Person.class, 2);  
  
Item newItem = new Item("Item from hibernate", person); // создаем item
person.getItems().add(newItem);  // и добавляем также этот предмет в person
  
session.save(newItem);  
  
session.getTransaction().commit();
```


Хорошей практикой считается добавлять с двух сторон. То есть здесь я создал предмет, и перед тем как сохранить его в БД я его также добавляю в список предметов этого person которому принадлежит предмет. Это нужно из-за того что Hibernate кэширует данные, и этот список предметов может быть не актуальным даже после вызова метода `save` 


Вот еще один пример:
```java
session.beginTransaction();  
  
Person person = new Person("Alex", 33);  
Item newItem = new Item( "Some item", person );  
person.setItems(new ArrayList<>(Collections.singletonList(newItem)));  
  
session.save( person );  
session.save( newItem );  
  
session.getTransaction().commit();
```


Здесь уже создается новый человек и новый предмет. В список предметов создается новый список и добавляется один предмет. И далее эти объекты сохраняются в БД.





## Удаление объектов


```java
session.beginTransaction();  
  
Person person = session.get(Person.class, 3);  
List<Item> items = person.getItems();  
  
// SQL  
for (Item item : items) {  
    session.remove(item);  
}  
  
// Не порождает SQL, но необходимо чтобы в кэше все было правильно  
person.getItems().clear();  
  
session.getTransaction().commit();
```



## Изменение объектов


```java
session.beginTransaction();  
  
Person person = session.get(Person.class, 4);  
Item item = session.get(Item.class, 1);  
  
item.getOwner().getItems().remove(item);  
  
// SQL  
item.setOwner(person);  
  
person.getItems().add(item);  
  
  
session.getTransaction().commit();
```


1. Получаем человека, которого хотим сделать владельцем предмета
2. Получаем предмет, которому хотим назначить нового владельца
3. `item.getOwner().getItems().remove(item);` - получаем владельца предмета, вызываем у него метод для получения всех его предметов и удаляем этот предмет. 
4. `item.setOwner(person);` - изменяем владельца
5. `person.getItems().add(item);` - к этому владельцу добавляем в список этот предмет





# Cascade types with collections


Допустим есть класс Company:
```java
@OneToMany(mappedBy = "company", cascade = CascadeTypes.ALL)
private Set<User> users;


public void addUser(User user) {
	users.add(user);
	user.setCompany(this);
}
```

Здесь есть связь один ко многим и мы устанавливаем CascadeTypes.ALL для того чтобы если мы добавляем компанию и в ней пользователя, этот пользователь также добавлялся в таблицу Users. 

И также я добавил метод addUser который добавляет пользователя в коллекцию и соответственно в таблицу Users, а также сразу устанавливаю ему компанию



## Entity equals and hashcode

```java
@EqualsAndHashCode(of = "id")
@ToString(of = "id")
```
Так методы `equals` и `hashCode` будут сравнивать именно по id



# Many To Many
https://javarush.com/quests/lectures/questhibernate.level13.lecture03


![[Pasted image 20250707235019.png]]





Я опущу в примерах существующие поля, зато добавлю новые. Вот как они будут выглядеть. Класс Employee:

```java
@Entity
@Table(name="employee")
class Employee {
   @Column(name="id")
   public Integer id;

   @ManyToMany(cascade = CascadeType.ALL)
   @JoinTable(name="employee_task",
	       joinColumns=  @JoinColumn(name="employee_id", referencedColumnName="id"),
           inverseJoinColumns= @JoinColumn(name="task_id", referencedColumnName="id") )
   private Set<EmployeeTask> tasks = new HashSet<EmployeeTask>();

}
```


И класс EmployeeTask:

```java
@Entity
@Table(name="task")
class EmployeeTask {
   @Column(name="id")
   public Integer id;

   @ManyToMany(cascade = CascadeType.ALL)
   @JoinTable(name="employee_task",
       	joinColumns=  @JoinColumn(name="task_id", referencedColumnName="id"),
       	inverseJoinColumns= @JoinColumn(name=" employee_id", referencedColumnName="id") )
   private Set<Employee> employees = new HashSet<Employee>();

}
```

Во-первых, там используется аннотация @JoinTable (не путать с @JoinColumn), которая описывает служебную таблицу employee_task.

Во-вторых, там описывается, что колонка task_id таблицы employee_task ссылается на колонку id таблицы task.

В-третьих, там говориться, что колонка employee_id таблицы employee_task ссылается на колонку id таблицы employee.

Мы фактически с помощью аннотаций описали какие данные содержатся в таблице employee_task и как Hibernate должен их интерпретировать.



### Пример: у меня есть три таблицы: Actors, Movies и Actors_Movies и нужно составить отношение многие-ко-многим


Сначала создаю класс Actor:
```java
@Getter  
@Setter  
@NoArgsConstructor  
@Entity  
@Table(name="Actor")  
public class Actor {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name="actor_id")  
    private int id;  
  
    private String name;  
  
    private int age;  
  
    @ManyToMany  
    @JoinTable(  
            name="Actor_Movie",  
            joinColumns = @JoinColumn(name="actor_id"),  
            inverseJoinColumns = @JoinColumn(name="movie_id")  
    )  
    private List<Movie> movies;  
}
```


В этом классе описываю отношение:
```java
@ManyToMany  
@JoinTable(  
	name="Actor_Movie",  
    joinColumns = @JoinColumn(name="actor_id"),  
    inverseJoinColumns = @JoinColumn(name="movie_id")  
) 
```


Аннотация `@JoinTable` показывает как именно нужно рассматривать это отношение.
Поле `name` означает таблицу, в которой описывается данное отношение.
Поле `joinColumns` обозначает id для сущности Actor в данной таблице.
Поле `inverseJoinColumns` обозначает id для сущности Movie.


Теперь можно создать класс `Movie`:
```java
@Getter  
@Setter  
@NoArgsConstructor  
@Entity  
@Table(name="Movie")  
public class Movie {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name="movie_id")  
    private int id;  
  
    private String name;  
  
    @Column(name="year_of_production")  
    private int yearOfProduction;  
  
    @ManyToMany(mappedBy = "movies")  
    private List<Actor> actors;  
}
```


Здесь отношение описывается проще:
```java
@ManyToMany(mappedBy = "movies")  
private List<Actor> actors; 
```

Здесь просто содержится ссылка на поле в классе Actor которое описывает данное отношение.



Вот пример с созданием одного фильма и двух актеров:
```java
session.beginTransaction();  
  
Movie movie = new Movie("Pulp fiction", 1994);  
Actor actor1 = new Actor("Harvey Keitel", 81);  
Actor actor2 = new Actor("Samuel L. Jackson", 72);  
  
movie.setActors(new ArrayList<>(List.of(actor1, actor2)));  
  
actor1.setMovies(new ArrayList<>(List.of(movie)));  
actor2.setMovies(new ArrayList<>(List.of(movie)));  
  
  
session.save(movie);  
  
session.save(actor1);  
session.save(actor2);  
  
session.getTransaction().commit();
```


Вывести всех актеров у фильма:
```java
session.beginTransaction();  
  
Movie movie = session.get(Movie.class, 1);  
System.out.println(movie.getActors());  
  
session.getTransaction().commit();
```


Создать новый фильм и связать его с существующим актером:
```java
session.beginTransaction();  
  
Movie movie = new Movie("Reservoir Dogs", 1992);  
Actor actor = session.get(Actor.class, 1);  
  
movie.setActors(new ArrayList<>(List.of(actor)));  
  
actor.getMovies().add(movie);  
  
session.save(movie);  
  
session.getTransaction().commit();
```


Удаление у актера фильма и у фильма этого актера соотвественно:
```java
session.beginTransaction();  
  
Actor actor = session.get(Actor.class, 2);  
System.out.println(actor.getMovies());  
  
Movie movieToRemove = actor.getMovies().get(0);  
  
actor.getMovies().remove(movieToRemove);  
movieToRemove.getActors().remove(actor);  
  
session.getTransaction().commit();
```


# One To One
https://javarush.com/quests/lectures/questhibernate.level13.lecture04


В случае же с Entity-классами могут быть варианты, которые описываются несколькими аннотациями:

- @Embedded
- Односторонний OneToOne
- Двусторонний OneToOne
- @MapsId


![[Pasted image 20250707132144.png]]





### Пример: есть сущность Person и сущность Passport, у них отношение один-к-одному. Вот как будет выстраиваться это отношение

Сначала создадим класс `Passport`:
```java
@Data  
@NoArgsConstructor  
@Entity  
@Table(name="passport")  
public class Passport implements Serializable {  
  
    @Id  
    @OneToOne()  
    @JoinColumn(name="person_id", referencedColumnName = "id")  
    private Person person;  
  
    @Column(name="passport_number")  
    private int passportNumber;  
  
  
    public Passport(Person person, int passportNumber) {  
        this.person = person;  
        this.passportNumber = passportNumber;  
    }  
  
    @Override  
    public String toString() {  
        return "Passport{" +  
                "passportNumber=" + passportNumber +  
                '}';  
    }  
}
```

Здесь первичный ключ это также внешний ключ, поэтому нужно имплементировать интерфейс `Serializable`

```java
@Id  
@OneToOne()  
@JoinColumn(name="person_id", referencedColumnName = "id")  
private Person person; 
```

Здесь поле помечается аннотацией `@OneToOne` и `@JoinColumn`. `name` показывает название этого поля, а `id` - поле в сущности Person на которое оно ссылается.


Теперь создадим класс `Person`:
```java
@Data  
@NoArgsConstructor  
@Entity  
@Table(name="person")  
public class Person {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    @Column(name="id")  
    private int id;  
  
    @Column(name="name")  
    private String name;  
  
    @Column(name="age")  
    private int age;  
  
    @OneToOne(mappedBy = "person")  
    @Cascade(org.hibernate.annotations.CascadeType.SAVE_UPDATE)  
    private Passport passport;  
  
    public Person(String name, int age) {  
        this.name = name;  
        this.age = age;  
    }  
  
    @Override  
    public String toString() {  
        return this.name + " " + this.age;  
    }  
}
```


В классе есть поле `passport` которое хранит паспорт:
```java
@OneToOne(mappedBy = "person")  
@Cascade(org.hibernate.annotations.CascadeType.SAVE_UPDATE)  
private Passport passport;  
```


Здесь просто указал на какое поле в классе `Passport` ссылается это поле и также настроил каскадирование, что при сохранении человека также будет сохраняться паспорт.


И вот так будет выглядеть сохранение человека и паспорта:
```java
session.beginTransaction();  
  
Person person = new Person("Test person", 23);  
Passport passport = new Passport(person, 12345);  
  
person.setPassport(passport);  
  
session.save(person);  
  
session.getTransaction().commit();
```


Также вместо того чтобы назначать в конструкторе человека, можно в методе у человека который назначает паспорт сразу задать назначение паспорту человека:
```java
public void setPassport(Passport passport) {  
    this.passport = passport;  
    passport.setPerson(this);  
}
```

И тогда сохранение будет выглядеть так:
```java
session.beginTransaction();  
  
Person person = new Person("Test person", 23);  
Passport passport = new Passport(12345);  
  
person.setPassport(passport);  
  
session.save(person);  
  
session.getTransaction().commit();
```





![[Pasted image 20250707135246.png]]



Для этого нужно изменить таблицу Passport, добавил туда id и сделав поле person_id уникальным:
```sql
create table passport (  
    id int primary key generated by default as identity ,  
    passport_number int not null ,  
    person_id int unique references person(id) on delete cascade  
);
```


Ну и поменять класс `Passport`:
```java
@Data  
@NoArgsConstructor  
@Entity  
@Table(name="passport")  
public class Passport {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private int id;  
  
    @Column(name="passport_number")  
    private int passportNumber;  
  
    @OneToOne  
    @JoinColumn(name="person_id", referencedColumnName = "id")  
    private Person person;  
  
    public Passport(int passportNumber) {  
        this.passportNumber = passportNumber;  
    }  
  
    @Override  
    public String toString() {  
        return "Passport{" +  
                "passportNumber=" + passportNumber +  
                '}';  
    }  
}
```







# HQL


### Знакомство с HQL

Ранее ты познакомился с Hibernate, а теперь я познакомлю тебя с **HQL**, он же **Hibernate Query Language**. Фактически, – это SQL переделанный под написание запросов в Hibernate. У него есть несколько ключевых отличий.

1. Использование **имени класса** вместо имени таблицы.
2. Использование **имени поля класса** вместо имени колонки таблицы.
3. **Необязательное** использование select.

Давай попросим Hibernate вернуть нам всех пользователей, которые есть у него в базе. Вот как будет выглядеть этот запрос:
```java
from User
```

`User` - это имя класса



Вот как будет выглядеть код в Java полностью:
```java
public List<User> getAllUsers() {
    try (Session session = sessionFactory.openSession()) {
            return session.createQuery("from User", User.class).list();
    }
}
```


В остальном HQL очень похож на SQL – в нем тоже есть операторы:
- `WHERE`
- `ORDER BY`
- `GROUP BY`
- `HAVING`



### Использование select

В HQL можно использовать `select`, когда тип данных результата не совпадает с типом указанным в `from`.

Например, мы хотим получить имена всех пользователей, которые есть в нашей таблице **user_data**, тогда нужно написать такой запрос:

```java
select name from User
```

Ну и полностью код выглядит так:
```java
public List<String> getUserNames() {
    try (Session session = sessionFactory.openSession()) {
            String hql = "select distinct u.name from User u where u.created > '2020-01-01'";
            Query<String> query = session.createQuery(hql , String.class);
            return query.list();
    }
}
```





## Изучаем класс Query

Лекция: https://javarush.com/quests/lectures/questhibernate.level10.lecture01


```java
public List<Employee> getAllEmployes() {
    try (Session session = sessionFactory.openSession()) {
            Query<Employee> query = session.createQuery("from Employee", Employee.class);
            return query.list();
    }
}
```

На самом деле, Query – это интерфейс и у него есть несколько реализаций на разные случаи. Но для простоты я буду продолжать называть его классом. Это, скажем так, класс в широком смысле – в терминах ООП.

Примечание. Раньше было два класса:

- **Query** для описания запроса.
- **TypedQuery** для описания запроса с заранее известным типом.




## Join в HQL

Лекция: https://javarush.com/quests/lectures/questhibernate.level10.lecture02


```java
@ManyToOne @JoinColumn(name="employee_id", nullable=true)
public Employee employee;
```

Аннотация `@ManyToOne` говорит Hibernate, что много сущностей **EmployeeTask** могут ссылаться на одну сущность **Employee**.

А аннотация `@JoinColumn` указывает имя колонки, из которой будет браться **id**. Вся остальная необходимая информация будет взята из аннотаций класса Employee.

Пример без JOIN

```java
@Entity @Table(name="task")
class EmployeeTask {
   @Column(name="id")
   public Integer id;

   @Column(name="name")
   public String name;

   @Column(name="employee_id")
   public Integer employeeId;

   @Column(name="deadline")
   public Date deadline;
}
```

Пример с JOIN
```java
@Entity @Table(name="task")
class EmployeeTask
{
   @Column(name="id")
   public Integer id;

   @Column(name="name")
   public String name;

   @ManyToOne @JoinColumn(name="employee_id", nullable=true)
   public Employee employee;

   @Column(name="deadline")
   public Date deadline;
}
```







# Ленивая загрузка



![[Pasted image 20250708165315.png]]


![[Pasted image 20250708165424.png]]



![[Pasted image 20250708165626.png]]





#### То есть при ленивой загрузке связанные сущности не подгружаются, если к ним не обращаются напрямую или их напрямую не подгружают, в отличие от не ленивой загрузки.


#### Также при не ленивой загрузки можно данные использовать вне сессии и вне транзакции, в отличие от ленивой. Чтобы вне сессии использовать данные при ленивой загрузке необходимо их подгрузить. Например, использовать команду `Hibernate.initialize()` в транзакции 

```java
Hibernate.initialize(person.getItems());
```








# Spring приложение с Hibernate

[[spring|Spring]]




В класс `SpringConfig` нужно добавить аннотации:
```java
@PropertySource("classpath:hibernate.properties")  
@EnableTransactionManagement
```


И подправить `DataSource`:
```java
@Bean  
public DataSource dataSource() {  
    DriverManagerDataSource dataSource = new DriverManagerDataSource();  
  
    dataSource.setDriverClassName(Objects.requireNonNull(environment.getProperty("hibernate.driver_class")));  
    dataSource.setUrl(environment.getProperty("hibernate.connection.url"));  
    dataSource.setUsername(environment.getProperty("hibernate.connection.username"));  
    dataSource.setPassword(environment.getProperty("hibernate.connection.password"));  
  
    return dataSource;  
}
```



И теперь вместо JdbcTemplate нужно добавить:
```java
private Properties hibernateProperties() {  
    Properties properties = new Properties();  
    properties.put("hibernate.dialect", environment.getRequiredProperty("hibernate.dialect"));  
    properties.put("hibernate.show_sql", environment.getRequiredProperty("hibernate.show_sql"));  
  
    return properties;  
}  
  
@Bean  
public LocalSessionFactoryBean sessionFactory() {  
    LocalSessionFactoryBean sessionFactory = new LocalSessionFactoryBean();  
    sessionFactory.setDataSource(dataSource());  
    sessionFactory.setPackagesToScan("ru.alishev.springcourse.models");  
    sessionFactory.setHibernateProperties(hibernateProperties());  
  
    return sessionFactory;  
}  
  
@Bean  
public PlatformTransactionManager hibernateTransactionManager() {  
    HibernateTransactionManager transactionManager = new HibernateTransactionManager();  
    transactionManager.setSessionFactory(sessionFactory().getObject());  
  
    return transactionManager;  
}
```



Теперь сначала нужно поменять класс Person чтобы он был сущностью Hibernate:
```java
@Getter  
@Setter  
@NoArgsConstructor  
@Entity  
@Table(name="person")  
public class Person {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Integer id;  
  
    @NotEmpty(message="Name should not be empty")  
    @Size(min=2, max=30, message = "Name should be between 2 and 30")  
    private String name;  
  
    @Min(value = 0, message="Age should be greater than zero")  
    private Integer age;  
  
    @NotEmpty(message="Email should not be empty")  
    @Email(message="Incorrect email!")  
    private String email;  
  
    public Person(String name, Integer age, String email) {  
        this.name = name;  
        this.age = age;  
        this.email = email;  
    }  
}
```


И поменять DAO, нужно внедрить объект `SessionFactory` ну и поменять методы.

Сначала поменяем метод index():
```java
@Transactional  
public List<Person> index() {  
    Session session = sessionFactory.getCurrentSession();  
  
    List<Person> people = session.createQuery("from Person", Person.class).getResultList();  
  
    return people;  
}
```


Хорошей практикой считается то, когда мы помечаем методы где только читаем `@Transactional(readOnly=true)`



Метод `show(Integer id)`:
```java
@Transactional(readOnly = true)  
public Optional<Person> show(Integer id) {  
    Session session = sessionFactory.getCurrentSession();  
  
    return Optional.ofNullable(session.get(Person.class, id));  
}
```


Метод `show(String email)`:
```java
@Transactional(readOnly = true)  
public Optional<Person> show(String email) {  
    Session session = sessionFactory.getCurrentSession();  
  
    return session.createQuery("from Person where email = :email", Person.class).setParameter("email", email).getResultList()  
            .stream().findFirst();  
  
}
```



Метод `save()`:
```java
@Transactional  
public void save(Person person) {  
    sessionFactory.getCurrentSession().persist(person);  
}
```


Метод `update()`:
```java
@Transactional  
public void update(Integer id, Person person) {  
    Person updatePerson = sessionFactory.getCurrentSession().get(Person.class, id);  
    if (updatePerson != null) {  
        updatePerson.setEmail(person.getEmail());  
        updatePerson.setName(person.getName());  
        updatePerson.setAge(person.getAge());  
    }  
}
```


Метод `delete()`:
```java
@Transactional  
public void delete(Integer id) {  
    Session session = sessionFactory.getCurrentSession();  
    Person person = session.get(Person.class, id);  
    if (person != null) {  
        session.remove(person);  
    }  
}
```








# Дата и время в Hibernate



Для обозначения даты и времени в Hibernate используется аннотация `@Temporal()`:
```java
@Temporal(TemporalType.DATE)
private Date date;
```

это для типа `Date`. Для типа `Timestamp` будет выглядеть так:
```java
@Temporal(TemporalType.TIMESTAMP)
```

Также нужно добавить аннотацию `@DateTimeFormat` и передать паттерн по которому нужно вводить дату:
```java
@Temporal(TemporalType.DATE)
@DateTimeFormat(pattern = "dd/MM/yyyy")
private Date dat
```








# Перечисления (Enum) в Hibernate




Чтобы сохранить Enum в БД с помощью Hibernate нужно использовать аннотацию `@Enumerated()` и в скобках указать способ сохранения.

- `EnumType.ORDINAL` - сохранение индексов 
- `EnumType.STRING` - сохранение названия





# Проблема N + 1


![[Pasted image 20250710162606.png]]



![[Pasted image 20250710162725.png]]


Пример с решением этой проблемы:
![[Pasted image 20250710163737.png]]








# Методы get() и load()



![[Pasted image 20250710164114.png]]




![[Pasted image 20250710164144.png]]




![[Pasted image 20250710164338.png]]










# Пагинация и сортировка 



![[Pasted image 20250710164742.png]]


![[Pasted image 20250710164909.png]]



![[Pasted image 20250710164934.png]]











# Аннотация @Transient



![[Pasted image 20250710165017.png]]



### Пример:

![[Pasted image 20250710165051.png]]







