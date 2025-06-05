# Statements


- **Statement** - это интерфейс, который используется для доступа к БД для общих целей. Он полезен когда мы используем статические SQL-выражения во время работы программы. Этот интерфейс не принимает никаких параметров.
- **PreparedStatement** - это интерфейс, который используется, когда мы хотим использовать SQL-выражения несколько раз. Он принимает параметры. В общем он используется для выполнения подготовленных SQL-запросов.
- **CallableStatement** - этот интерфейс полезен в случае, когда мы хотим получить доступ к различным процедурам БД. Он также принимает параметры.


Пример Statement:
```java
try {
            statement = connection.createStatement();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            /*Do some job...*/
        }
```

Пример PreparedStatement:
```java
 try {
            String SQL = "Update developers SET salary WHERE specialty = ?";
            preparedStatement = connection.prepareStatement(SQL);
        }catch (SQLException e){
            e.printStackTrace();
        }finally {
            /*do some job...*/
        }
```





## Введение в JDBC

[[JDBC|Подробное рассмотрение JDBC]]



**JDBC (Java DataBase Connectivity)** - это API, предоставляющий низкоуровневые инструменты для работы с БД через SQL-запросы

#### Как работает JDBC?
- Открываем соединение с БД
- Создаем SQL-запрос и выполняем его
- Получаем `ResultSet` для `SELECT` запроса и `int` для `UPDATE`, `INSERT`, `DELETE`
- Закрываем соединение


```java
import java.sql.*;

public class JDBCDemo {
    public static void main(String[] args) {
        String url = "jdbc:mysql://localhost:3306/testdb";
        String user = "root";
        String password = "1234";

        try (Connection conn = DriverManager.getConnection(url, user, password)) {
            String sql = "SELECT * FROM users WHERE id = ?";
            PreparedStatement stmt = conn.prepareStatement(sql);
            stmt.setInt(1, 5);

            ResultSet rs = stmt.executeQuery();
            while (rs.next()) {
                System.out.println(rs.getString("name"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```


✅ **Плюсы JDBC**:  
✔ Полный контроль над SQL-запросами.  
✔ Подходит для небольших проектов, где нужен **прямой доступ к БД**.  
✔ Быстрее, чем ORM (нет лишних абстракций).

❌ **Минусы JDBC**:  
✖ Много **рутинного кода**: открытие соединений, обработка исключений, закрытие ресурсов.  
✖ Нет автоматического маппинга между объектами Java и таблицами БД.  
✖ Повышенный риск **SQL-инъекций**, если не использовать `PreparedStatement`.


Ну и нужно разобрать что такое Hibernate


### Hibernate

**Hibernate** - это ORM-фреймворк, который позволяет работать с БД через Java-объекты а не через SQL-запросы

### 🔹 **Как работает Hibernate?**

1. **Создаётся класс-модель (Entity)**, соответствующий таблице.
2. **Hibernate сам генерирует SQL-запросы** на основе объекта.
3. **Выполняем CRUD-операции через методы Hibernate**, а не через SQL.


Пример кода с Hibernate
1. Определяем сущность (Entity)
```java
import jakarta.persistence.*;

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "name")
    private String name;

    public User() {}

    public User(String name) {
        this.name = name;
    }

    public Long getId() { return id; }
    public String getName() { return name; }
}
```

2. Работа с Hibernate (CRUD)
```java
import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;

public class HibernateDemo {
    public static void main(String[] args) {
        SessionFactory factory = new Configuration().configure("hibernate.cfg.xml").buildSessionFactory();
        Session session = factory.openSession();

        Transaction tx = session.beginTransaction();
        User user = new User("Иван");
        session.save(user); // Hibernate автоматически создаст SQL INSERT

        tx.commit();
        session.close();
        factory.close();
    }
}
```



### Отличия JDBC и Hibernate

| Функция                | JDBC                                           | Hibernate                              |
| ---------------------- | ---------------------------------------------- | -------------------------------------- |
| **Подход**             | Работа напрямую с SQL                          | Объектно-реляционное отображение (ORM) |
| **SQL-запросы**        | Нужно писать вручную                           | Генерируются автоматически             |
| **Количество кода**    | Много рутинного кода                           | Минимум кода (всё через объекты)       |
| **Производительность** | Быстрее, но сложнее                            | Чуть медленнее, но удобнее             |
| **Безопасность**       | SQL-инъекции возможны без `PreparedStatement`  | Защищён от SQL-инъекций                |
| **Гибкость**           | Полный контроль над SQL                        | Меньше контроля, но удобнее            |
| **Работа с БД**        | Работает с **любой БД**, требует написания SQL | Требует настройки под конкретную БД    |
| **Транзакции**         | Обрабатываются вручную                         | Встроенная поддержка                   |
| **Кэширование**        | Нет                                            | Есть встроенное кэширование            |


### Когда использовать что?

✅ **Используйте JDBC, если**:

- Нужно **максимальная производительность**.
- Проект **маленький**, и ORM не оправдан.
- Важно **полный контроль над SQL-запросами**.

✅ **Используйте Hibernate, если**:

- Проект **крупный** и сложный.
- Нужно **минимизировать SQL-код**.
- Важна **гибкость и удобство** при работе с БД.