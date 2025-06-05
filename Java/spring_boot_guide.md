
## SpringBoot Project

`@RestController` - означает что класс будет принимать http запросы (я поставил над классом EmployeeController)
`@GetMapping` - аннотация ставится над методом который принимает GET-запросы (этот метод должные быть в классе помеченным @RestController)


Запустил docker-container:
```bash
docker run --name spring-boot-postgres -p 5432:5432 -e POSTGRES_PASSWORD=root -d postgres
```




Добавил Базу Данных, а в properties прописал:
```properties
spring.datasource.url=jdbc:postgresql://localhost:5433/postgres  
spring.datasource.username=postgres  
spring.datasource.password=root  
spring.jpa.hibernate.ddl-auto=create-drop
spring.jpa.show-sql=true
```



Потом обновил класс Employee сделал его сущностью в БД:
```java
@Entity  
@Table(name="employee")  
@Data  
@NoArgsConstructor  
@AllArgsConstructor  
@Builder  
public class Employee {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
    private String name;  
    private String email;  
    private LocalDate birthDate;  
    private Integer age;  
    private Integer salary;  
}
```



Далее создал интерфейс EmployeeRepository:
```java
public interface EmployeeRepository extends JpaRepository<Employee, Long>
```

### READ

Потом я добавил EmployeeRepository в EmployeeService и поменял метод для поиска всех сотрудников:
```java
@Service  
@RequiredArgsConstructor  
public class EmployeeService {  
  
    private final EmployeeRepository employeeRepository;  
  
  
    public List<Employee> getAllEmployees() {  
        return employeeRepository.findAll();  
    }  
  
}
```



Далее создал EmployeeConfig который сохраняет данные в базу в самом начале работы программы:
```java
@Configuration  
public class EmployeeConfig {  
  
  
    @Bean  
    CommandLineRunner commandLineRunner(  
            EmployeeRepository repository  
    ) {  
        return args -> {  
            var employeeList = List.of(  
                    new Employee(  
                            null,  
                            "Vasya",  
                            "vasya@gmail.com",  
                            LocalDate.of(2000, 10, 1),  
                            25,  
                            10000  
                    ),  
                    new Employee(  
                            null,  
                            "Pasha",  
                            "pasha@gmail.com",  
                            LocalDate.of(2003, 11, 12),  
                            22,  
                            20000  
                    )  
            );  
            repository.saveAll(employeeList);  
        };  
    }  
  
}
```



Немного поменял сущность Employee так, чтобы поле age не сохранялось в БД и вычислялось на основе даты:

```java
@Entity  
@Table(name="employee")  
@Getter  
@Setter  
@NoArgsConstructor  
@AllArgsConstructor  
@Builder  
public class Employee {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
    private String name;  
    private String email;  
    private LocalDate birthDate;  
    //private Integer age;  
    private Integer salary;  
  
  
    @Transient  
    public Integer getAge() {  
        if (birthDate == null) {  
            return null;  
        }  
        return Period.between(birthDate, LocalDate.now()).getYears();  
    }  
}
```


### CREATE

Добавил в класс EmployeeController метод createEmployee:
```java
@PostMapping  
public Employee createEmployee(  
        @RequestBody Employee employee  
) {  
    return employeeService.createEmployee(employee);  
}
```

Но перед этим нужно создать метод createEmployee в EmployeeService:
```java
public Employee createEmployee(Employee employee) {  
    if (employee.getId() != null) {  
        throw new IllegalArgumentException("Employee id already exists");  
    }  
    // unique email  
    if (employeeRepository.findByEmail(employee.getEmail()).isPresent()) {  
        throw new IllegalArgumentException("Employee email already exists");  
    }  
    // salary > 5000  
    if (employee.getSalary() <= 5000) {  
        throw new IllegalArgumentException("Salary less than 5000");  
    }  
    return employeeRepository.save(employee);  
}
```


А еще нужно создать метод createByEmail в employeeRepository:
```java
Optional<Employee> findByEmail(String email);
```

### DELETE

Создал метод deleteEmployee в EmployeeController:
```java
@DeleteMapping("/employeeId")  
public void deleteEmployee(  
        @PathVariable("employeeId") Long id  
) {  
    employeeService.deleteEmployee(id);  
}
```

Но перед этим нужно создать метод deleteEmployee в EmployeeService:
```java
public void deleteEmployee(Long id) {  
    if (employeeRepository.findById(id).isEmpty()) {  
        throw new IllegalArgumentException("Employee not found");  
    }  
    employeeRepository.deleteById(id);  
}
```



### UPDATE

Создал метод updateEmployee в EmployeeController:
```java
@Transactional  
@PutMapping("/employeeId")  
public void updateEmployee(  
        @PathVariable("employeeId") Long id,  
        @RequestParam(value = "email", required = false) String email,  
        @RequestParam(value = "salary", required = false) Integer salary  
) {  
    employeeService.updateEmployee(id, email, salary);  
}
```

И метод updateEmployee в EmployeeService:
```java
public void updateEmployee(Long id, String email, Integer salary) {  
    Employee employee = employeeRepository.findById(id)  
            .orElseThrow(() -> new IllegalArgumentException("Employee not found"));  
  
    Optional<Employee> existingByEmail = employeeRepository.findByEmail(email);  
    if (existingByEmail.isPresent() && !existingByEmail.get().getId().equals(id) && email != null) {  
        throw new IllegalArgumentException("Email already in use by another employee");  
    }  
    employee.setEmail(email);  
  
    if (salary <= 5000) {  
        throw new IllegalArgumentException("Salary must be greater than 5000");  
    }  
    employee.setSalary(salary);  
}
```



### Сборка проекта и  запуск

Сначала я сделал maven clean и package

В результате у меня в папке target появился jar-файл

Я его запустил при помощи команды `java -jar`



Далее создал в папке target файл Dockerfile:
```bash
FROM openjdk:21
WORKDIR /app
COPY spring-boot-guide-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

Далее создал docker-compose.yml:
```yml
version: '3.8'  
  
services:  
  db:  
    image: postgres  
    container_name: my-postgres  
    environment:  
      POSTGRES_USER: postgres  
      POSTGRES_PASSWORD: root  
      POSTGRES_DB: postgres  
    ports:  
      - "5433:5432"  
    networks:  
      - app-net  
  
  app:  
    build: .  
    container_name: my-spring-app  
    depends_on:  
      - db  
    ports:  
      - "8080:8080"  
    environment:  
      SPRING_DATASOURCE_URL: jdbc:postgresql://db:5432/postgres  
      SPRING_DATASOURCE_USERNAME: postgres  
      SPRING_DATASOURCE_PASSWORD: root  
      SPRING_JPA_PROPERTIES_HIBERNATE_DIALECT: org.hibernate.dialect.PostgreSQLDialect  
    networks:  
      - app-net  
  
networks:  
  app-net:
```

Собрал контейнер:
```bash
sudo docker-compose build
```

И запустил:
```bash
sudo docker-compose up
```



