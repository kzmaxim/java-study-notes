
![[Pasted image 20250531144348.png]]


Эти 3 строчки сгенерируют полноценный DAO 



![[Pasted image 20250531144509.png]]


![[Pasted image 20250531144719.png]]


![[Pasted image 20250531145018.png]]





Чтобы приступить к работе с Spring Data JPA, нужно сначала подключить пакет `spring-data-jpa`, а потом немного подправить SpringConfig.

Для начала нужно повесить аннотацию `@EnableJpaRepositories("com.maxim.repositories")`

А потом добавить entityManagerFactory и transactionalManager:
```java
@Bean  
public LocalContainerEntityManagerFactoryBean entityManagerFactory() {  
    final LocalContainerEntityManagerFactoryBean em = new LocalContainerEntityManagerFactoryBean();  
    em.setDataSource(dataSource());  
    em.setPackagesToScan("com.maxim.model");  
  
    final HibernateJpaVendorAdapter vendorAdapter = new HibernateJpaVendorAdapter();  
    em.setJpaVendorAdapter(vendorAdapter);  
    em.setJpaProperties(hibernateProperties());  
  
    return em;  
}  
  
@Bean  
public PlatformTransactionManager transactionManager() {  
    JpaTransactionManager transactionManager = new JpaTransactionManager();  
    transactionManager.setEntityManagerFactory(entityManagerFactory().getObject());  
  
    return transactionManager;  
}
```



И после этого можно создавать репозиторий:
```java
@Repository  
public interface PeopleRepository extends JpaRepository<Person, Integer> {  
}
```

И эти 3 строчки кода сгенерировали основные CRUD методы 


Теперь нужно создать сервис:
```java
@Service  
@Transactional(readOnly = true)  
public class PeopleService {  
  
    private final PeopleRepository peopleRepository;  
  
    @Autowired  
    public PeopleService(PeopleRepository peopleRepository) {  
        this.peopleRepository = peopleRepository;  
    }
```


В сервисе прописывается вся бизнес-логика и работа с данными, а именно работа с репозиториями.



И теперь создаем методы.

Метод `findAll()`:
```java
public List<Person> findAll() {  
    return peopleRepository.findAll();  
}
```


Метод `findOne()`:
```java
public Person findOne(int id) {  
    Optional<Person> person = peopleRepository.findById(id);  
      
    return person.orElse(null);  
}
```


Метод `save()`:
```java
@Transactional  
public void save(Person person) {  
    peopleRepository.save(person);  
}
```


Метод `update()`:
```java
@Transactional  
public void update(int id, Person updatedPerson) {  
    updatedPerson.setId(id);  
    peopleRepository.save(updatedPerson);  
}
```


Метод `delete()`:
```java
@Transactional  
public void delete(int id) {  
    peopleRepository.deleteById(id);  
}
```








# Кастомные запросы в Spring Data JPA




![[Pasted image 20250709234320.png]]




![[Pasted image 20250709234446.png]]




### Например, есть два класса Person и Item у них отношение один-ко-многим. И у Item есть поле person которое ссылается на Person

В репозитории `ItemsRepository` можно прописать метод который будет искать item по этому полю person:
```java
@Repository  
public interface ItemsRepository extends JpaRepository<Item, Integer> {  
    List<Item> findByPerson(Person person); // ищет по полю person 
}
```


Или вот пример нескольких методов:
```java
@Repository  
public interface PeopleRepository extends JpaRepository<Person, Integer> {  
    List<Person> findByName(String name);  
  
    List<Person> findByNameOrderByAge(String name);  
  
    List<Person> findByEmail(String email);  
  
    List<Person> findByNameStartingWith(String name);  
  
    List<Person> findByNameOrEmail(String name, String email);  
}
```





