# Inversion Of Control, Dependency Injection



![[Pasted image 20250421195613.png]]


Файл UserService:
```java
public class UserService {  
    private final UserRepository userRepository;  
  
    public UserService(UserRepository userRepository) {  
        this.userRepository = userRepository;  
    }  
  
}
```

Файл UserRepository от которого зависит UserService:
```java
public class UserRepository {  
    private final ConnectionPool pool;  
  
    public UserRepository(ConnectionPool pool) {  
        this.pool = pool;  
    }  
  
}
```

Файл ConnectionPool от которого зависит UserRepository:
```java
public class ConnectionPool {  
}
```

То есть зависимости находятся на уровне полей класса




![[Pasted image 20250421200746.png]]

![[Pasted image 20250421200838.png]]


![[Pasted image 20250421201322.png]]

## IoC Container

![[Pasted image 20250421201555.png]]


**Bean** - объект со всеми необходимыми зависимостями, который был создан с помощью **IoC Container** (Controller, Service, Repository)

![[Pasted image 20250604000019.png]]

![[Pasted image 20250604000543.png]]


![[Pasted image 20250604000646.png]]



![[Pasted image 20250421201828.png]]


![[Pasted image 20250421202043.png]]



Добавить Spring Core через Maven:
```xml
<dependencies>  
    <!-- https://mvnrepository.com/artifact/org.springframework/spring-core -->  
    <dependency>  
        <groupId>org.springframework</groupId>  
        <artifactId>spring-core</artifactId>  
        <version>6.2.6</version>  
    </dependency>  
</dependencies>
```
И Spring Context:
```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-context -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.2.6</version>
</dependency>
```





# ApplicationContext

Это IoC контейнер который управляет бинами (зависимостями)


![[Pasted image 20250603202553.png]]

![[Pasted image 20250603202705.png]]

Пример:
```java
ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("application.xml");  
System.out.println(context.getBean(ConnectionPool.class));
```

Но чтобы все работало надо записать этот класс в xml:
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">  
  
  
    <bean class="com.spring.database.pool.ConnectionPool"/>  
</beans>
```


Но в таком случае с помощью метода `getBean` мы получаем тип Object, чтобы такого не было нужно вторым параметром указать ожидаемый класс:
```java
context.getBean("pool1", ConnectionPool.class)
```
Я сюда также добавил обращение к бину через id, то есть:
```xml
<bean id="pool1" class="com.spring.database.pool.ConnectionPool"/>  
```

Кстати, при такой записи вызывается конструктор по умолчанию у `ConnectionPool`


# Dependency Injection

![[Pasted image 20250421201322.png]]

![[Pasted image 20250604182249.png]]


![[Pasted image 20250604182353.png]]




### Constructor Injection

Чтобы не вызывался конструктор по умолчанию, нужно создать конструктор в классе, а потом сконфигурировать его в xml-файле

Класс ConnectionPool:
```java
public class ConnectionPool {  
    private final String username;  
    private final String password;  
    private final List<Object> list;  
    private final Map<String, String> map;  
  
    public ConnectionPool(String username, String password, List<Object> list, Map<String, String> map) {  
        this.username = username;  
        this.password = password;  
        this.list = list;  
        this.map = map;  
    }  
}
```

И конфигурация в xml:
```xml
<bean class="com.spring.database.pool.ConnectionPool">  
    <constructor-arg value="postgres"/> username 
    <constructor-arg value="1234"/>   password
    <constructor-arg>  List<Object> list
        <list>  
            <value>--arg1=value1</value>  значение 1 у списка
            <value>--arg2=value2</value>  значение 2 у списка
        </list>  
    </constructor-arg>  
    <constructor-arg>  Map<String, String> map
        <map>  
            <entry key="url" value="postgresurl"/>  значение 1 у мапы
            <entry key="password" value="1234"/>   значение 2 у мапы
        </map>  
    </constructor-arg>  
</bean>
```

Значения конструктора помещаются в `<constructor-arg>`



Еще один пример:

Есть класс `MusicPlayer` который использует интерфейс `Music`:
```java
public class MusicPlayer {  
    private Music music;  
  
    // IoC  
    public MusicPlayer(Music music) {  
        this.music = music;  
    }  
  
    public void playMusic() {  
        System.out.println("Playing music: " + music.getSong());  
    }  
}
```

И вот как можно внедрить зависимости при помощи xml:
```xml
<bean id="musicBean"  
      class="com.maxim.MetallMusic">  
</bean>  
  
<bean id="musicPlayer" class="com.maxim.MusicPlayer">  
    <constructor-arg ref="musicBean" />  
</bean>
```






## Factory Method Injection


![[Pasted image 20250604194631.png]]


![[Pasted image 20250604194659.png]]




```xml
<bean class="com.spring.database.pool.ConnectionPool">  
    <constructor-arg value="postgres"/>  
    <constructor-arg value="1234"/>  
    <constructor-arg>  
        <list>  
            <value>--arg1=value1</value>  
            <value>--arg2=value2</value>  
            <ref bean="conversionService"/>  // ссылка на ConversionService
        </list>  
    </constructor-arg>  
    <constructor-arg>  
        <map>  
            <entry key="url" value="postgresurl"/>  
            <entry key="password" value="1234"/>  
            <entry key="service" value-ref="conversionService"/>  // ссылка на ConversionService
        </map>  
    </constructor-arg>  
</bean>
```

Можно внедрять зависимости при помощи ссылок `ref`, `value-ref`



А вот пример фабричного метода:
```java
public class UserRepository {  
    private final ConnectionPool pool;  
  
    private UserRepository(ConnectionPool pool) {  
        this.pool = pool;  
    }  
  
    public static UserRepository of(ConnectionPool pool) {  
        return new UserRepository(pool);  
    }  
}
```


А потом в xml-файле нужно создать бин:
```xml
<bean id="userRepository" class="com.spring.database.repository.UserRepository" factory-method="of"> // указываем фабричный метод 
    <constructor-arg ref="pool"/>  
</bean>
```




#### Пример:

Для того чтобы работал factory method нужно сделать конструктор приватным и потом собственно создать этот factory method:

```java
private ClassicalMusic() {}  // приватный конструктор
  
public static ClassicalMusic getInstance() {  // фабричный метод
    return new ClassicalMusic();  
}
```

И теперь указываем спрингу что при создании бина нужно использовать именно фабричный метод:
```xml
<bean id="classicalMusicBean"  
      class="com.maxim.ClassicalMusic"  
      factory-method="getInstance"  
      init-method="doMyInit"  
      destroy-method="doMyDestroy">  <?-- указываем фабричный метод /--?>
</bean>
```


## Property Injection



Суть заключается в том что мы создаем сеттеры для полей класса, а в xml внедрение происходит при помощи тэга `property` 


![[Pasted image 20250604183331.png]]


![[Pasted image 20250604183513.png]]


![[Pasted image 20250604183555.png]]



Пример внедрения зависимости через setter:
```xml
<bean id="musicBean"  
      class="com.maxim.MetallMusic">  
</bean>  
  
<bean id="musicPlayer" class="com.maxim.MusicPlayer">  
    <property name="music" ref="musicBean" />  
</bean>
```

Но в данном случае у бина `MusicPlayer` должен быть пустой конструктор 


Чтобы внедрить значения из файла `.properties` нужно сначала этот файл подключить:
```xml
<context:property-placeholder location="classpath:musicPlayer.properties"/>
```

Ну а чтобы внедрить данные из этого файла нужно использовать `${}`:
```xml
<bean id="musicPlayer" class="com.maxim.MusicPlayer">  
    <property name="music" ref="musicBean" />  
  
    <property name="name" value="${musicPlayer.name}"/>  
    <property name="volume" value="${musicPlayer.volume}"/>  
</bean>
```


Пример внедрения зависимостей если зависимость - список:
```xml
<bean id="musicPlayer" class="com.maxim.MusicPlayer">  
    <property name="musicList">  
        <list>  
            <ref bean="metallMusicBean"/>  
            <ref bean="classicalMusicBean"/>  
            <ref bean="rockMusicBean"/>  
        </list>  
    </property>  
  
    <property name="name" value="${musicPlayer.name}"/>  
    <property name="volume" value="${musicPlayer.volume}"/>  
</bean>
```


## Bean Scopes

Bean Scopes определяют как и когда создаются экземпляры бинов

![[Pasted image 20250604191434.png]]

Bean Scopes:
- `Singleton` - один экземпляр бина создается для всего контекста Spring. Это подходит для бинов, состояние которых должно быть единым для всех компонентов приложения
- `Prototype` - при запросе к контейнеру создается новый экземпляр бина. Такие бины не сохраняются с ApplicationContext и не управляются им. Чтобы создать такой бин нужно над классов указать `@Scope("prototype")`
```java
@Service
@Scope("prototype")
public class UserService
```

- `Request` 
- `Session`
- `WebSocket`

Spring создает только один экземпляр Bean одного и того же типа, то есть бины - синглтоны


![[Pasted image 20250424192548.png]]


- Common
  -  `singleton` - создается один экземпляр бина
![[Pasted image 20250604191520.png]]
![[Pasted image 20250604191736.png]]


  - `prototype` - каждый раз создается новый объект
![[Pasted image 20250604191955.png]]
![[Pasted image 20250604192106.png]]

- Web (WebApplicationContext)
  -  `request` - для каждого запроса создается новый бин
  - `session` - на каждую сессию создается новый бин
  - `application` - на сервлет создается бин
  - `websocket` 

Пример:
```xml
<bean id="pool" class="com.spring.database.pool.ConnectionPool" scope="prototype">
```









# Bean Lifecycle


![[Pasted image 20250502223741.png]]



![[Pasted image 20250604192833.png]]

![[Pasted image 20250604193104.png]]


![[Pasted image 20250604193159.png]]


#### Пример:

Создал два метода в бине `ClassicalMusic`:
```java
public void doMyInit() {  
    System.out.println("Classical Music init");  
}  
public void doMyDestroy() {  
    System.out.println("Classical Music destroy");  
}
```


И указал их в xml файле при создании бина:
```xml
<bean id="classicalMusicBean"  
      class="com.maxim.ClassicalMusic"  
    init-method="doMyInit"  
    destroy-method="doMyDestroy">  
</bean>
```

Вывод:
```
Classical Music init
Bohemian Rhapsody
Classical Music destroy
```



![[Pasted image 20250604193816.png]]


![[Pasted image 20250604193922.png]]




## Lifecycle Callbacks


![[Pasted image 20250424194223.png]]




![[Pasted image 20250424192137.png]]



`Initialization Callbacks` - после того как мы вызвали setter'ы, мы может также вызвать другие методы для дополнительной инициализации или что-нибудь подкрутить


Это можно сделать тремя способами:
1. @PostConstruct - предпочтительный
2. afterPropertiesSet() - InitizlizingBean
3. init-method - xml


`Destruction Callbacks` - когда мы хотим очистить все ресурсы после использования бина

Это можно сделать тремя способами:
1. @PreDestroy
2. destroy() - DisposableBean
3. detsroy-method - xml



## Injection From Properties Files



Вот один из способов указать properties:
```xml
<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">  
    <property name="locations" value="classpath:application.properties"/>  
</bean>
```


Но лучше использовать другой способ. Сначала нужно подправить шапку у xml-файла:
```xml
xmlns:context="http://www.springframework.org/schema/context"  
xsi:schemaLocation="http://www.springframework.org/schema/beans  
http://www.springframework.org/schema/beans/spring-beans.xsd  
http://www.springframework.org/schema/context  
http://www.springframework.org/schema/context/spring-context-4.0.xsd">
```

А потом просто дописать:
```xml
<context:property-placeholder location="classpath:application.properties"/>
```




## BeanFactoryPostProcessor



![[Pasted image 20250424220644.png]]









# Annotation-Based Configuration


![[Pasted image 20250604200509.png]]

![[Pasted image 20250604200558.png]]

![[Pasted image 20250604200614.png]]

![[Pasted image 20250604200715.png]]

Чтобы Spring знал где именно сканировать классы, нужно в конфигурации указать:
```xml
<context:component-scan base-package="com.maxim"/>
```

Теперь создания бина выглядит примерно так:
```java
@Component("classicalMusicBean")  
public class ClassicalMusic implements Music {  
    @PostConstruct  
    public void doMyInit() {  
        System.out.println("Classical Music init");  
    }  
    @PreDestroy  
    public void doMyDestroy() {  
        System.out.println("Classical Music destroy");  
    }  
    @Override  
    public String getSong() {  
        return "Bohemian Rhapsody";  
    }  
}
```



Для использования необходимых аннотация нужно сначала загрузить необходимую библиотеку Jakarta Annotation API
```xml
<dependency>  
    <groupId>jakarta.annotation</groupId>  
    <artifactId>jakarta.annotation-api</artifactId>  
    <version>3.0.0</version>  
</dependency>
```

Теперь init-method можно подключать не с помощь xml-файла а с помощью аннотации `@PostConstruct`:
```java
@PostConstruct  
public void init() {  
    System.out.println("Init ConnectionPool");  
}
```


И также с методом destroy но уже используя аннотацию `@PreDestroy`:
```java
@PreDestroy  
public void destroy() {  
    System.out.println("Destroy ConnectionPool");  
}
```


А в сам xml-файл нужно добавить бин:
```xml
<bean class="org.springframework.context.annotation.CommonAnnotationBeanPostProcessor"/>
```

Но вместо создания бина можно просто прописать в шапке xml:
```xml
<context:annotation-config/>
```


![[Pasted image 20250426150726.png]]



## Bean Post Processor


![[Pasted image 20250426202009.png]]


**`BeanPostProcessor`** — это специальный механизм в Spring, который позволяет вмешиваться в процесс создания бина **до** и **после** его инициализации.

То есть, когда Spring создаёт объект-бин:

- до того как бин будет готов к использованию, можно что-то изменить в нём (например, установить дополнительные параметры, обернуть его прокси);
    
- после того как бин проинициализирован (выполнены сеттеры, методы `@PostConstruct` и так далее), тоже можно изменить его или добавить дополнительное поведение.
    

**Пример в жизни:**  
Допустим, ты строишь машину. После сборки (инициализации) кто-то проверяет её — если что-то не так, докручивает гайки или настраивает электронику. Вот `BeanPostProcessor` — это как инженер, который проверяет машину до и после сборки.

---

### Теперь по схеме:

Схема называется **"Bean Lifecycle"** — жизненный цикл бина.  
Разберем её пошагово:

1. **Bean Definitions**
    
    - Сначала у Spring есть описание бинов — как их создавать (`Bean Definitions`).
        
2. **Bean Factory Post Processors**
    
    - На этом этапе **BeanFactoryPostProcessor** может изменить определения бинов **до** их создания.  
        (Например, изменить конфигурацию.)
        
3. **Sorted Bean Definitions**
    
    - Бины сортируются по зависимостям и другим параметрам.
        
4. **Bean constructor called**
    
    - Для каждого бина вызывается его **конструктор**.
        
5. **Setters called**
    
    - Вызываются **сеттеры** для установки зависимостей (`@Autowired`, зависимости через поля или методы).
        
6. **BPP Before Initialization (BeanPostProcessor Before Initialization)**
    
    - Тут Spring передаёт бин через **BeanPostProcessor** перед инициализацией.
        
    - Здесь можно вмешаться и изменить бин (например, обернуть его в прокси).
        
7. **Initialization callbacks**
    
    - Срабатывают различные инициализационные методы:
        
        - `@PostConstruct`
            
        - `afterPropertiesSet()` из интерфейса `InitializingBean`
            
        - Метод `init-method` из XML-конфигурации.
            
8. **BPP After Initialization (BeanPostProcessor After Initialization)**
    
    - Снова Spring передает бин через **BeanPostProcessor**, но уже **после** инициализации.
        
    - Тут можно, например, ещё раз модифицировать бин.
        
9. **Beans**
    
    - Теперь бин готов к использованию. Контейнер хранит его для работы.
        
10. **Destruction callbacks (только для Singleton Beans)**
    
    - Когда контейнер закрывается (например, при остановке приложения), для каждого **Singleton** бина вызываются методы разрушения:
        
        - `@PreDestroy`
            
        - `destroy()` из интерфейса `DisposableBean`
            
        - Метод `destroy-method` из XML-конфигурации.
            

---

### Краткое резюме:

- `BeanPostProcessor` участвует дважды: **до инициализации** и **после инициализации**.
    
- Он нужен для того, чтобы **изменять или оборачивать** бины.
    
- Сначала создается бин через конструктор, затем вызываются сеттеры, потом инициализация (`@PostConstruct` и т.д.), потом бин можно ещё раз обработать через `BeanPostProcessor`, и в конце, при завершении приложения, бин разрушается.




## Custom Bean Post Processor


### Зачем создавать свой `BeanPostProcessor`?

Ты создаёшь `BeanPostProcessor`, когда хочешь:

- автоматически **изменять** или **оборачивать** все бины определённого типа;
    
- добавлять **дополнительное поведение** без изменения самих классов бинов;
    
- делать **автоматическую обработку аннотаций**;
    
- **оборачивать** бины в прокси для добавления функционала (например, логирование всех методов).
    

### Примеры реальных случаев:

|Ситуация|Как помогает `BeanPostProcessor`|
|---|---|
|Нужно, чтобы все сервисы логировали вызовы методов|Автоматически оборачиваешь все сервисы в прокси-объекты|
|Нужно, чтобы все бины с аннотацией `@MySpecialAnnotation` выполняли какое-то действие при старте|Через пост-процессор находишь такие бины и что-то с ними делаешь|
|Нужно добавлять значения в поля автоматически|Читаешь аннотации, подставляешь значения|

---

### Как создать свой `BeanPostProcessor`?

Очень просто! Надо:

1. Создать обычный Java класс.
    
2. Реализовать интерфейс `BeanPostProcessor`.
    
3. Переопределить два метода:
    
    - `postProcessBeforeInitialization`
        
    - `postProcessAfterInitialization`
        

Вот базовый пример:
```java
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.stereotype.Component;

@Component
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) {
        // Этот метод вызывается ПЕРЕД инициализацией (@PostConstruct и т.д.)
        System.out.println("Before Initialization: " + beanName);
        return bean; // возвращаем бин дальше без изменений (или модифицированный)
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) {
        // Этот метод вызывается ПОСЛЕ инициализации
        System.out.println("After Initialization: " + beanName);
        return bean; // можно вернуть модифицированный бин
    }
}
```


### Что происходит с таким `BeanPostProcessor`?

- Spring сам находит твой `MyBeanPostProcessor`, потому что он помечен как `@Component`.
    
- Когда Spring создаёт каждый бин, он **автоматически** вызывает эти два метода.
    
- Ты можешь:
    
    - Проверить имя бина или его тип.
        
    - Изменить бин.
        
    - Обернуть его в прокси или добавить поведение.
        

---

### Мини-схема процесса:
```java
Создание бина -> 
BeanPostProcessor (before initialization) -> 
@Init методы (@PostConstruct и др.) -> 
BeanPostProcessor (after initialization) -> 
Готовый бин
```



### Маленький жизненный пример:

Допустим, ты хочешь, чтобы все сервисы автоматически писали в лог "Я готов!" после создания.
```java
@Override
public Object postProcessAfterInitialization(Object bean, String beanName) {
    if (bean instanceof MyService) {
        System.out.println(beanName + " is ready to serve!");
    }
    return bean;
}
```



### Короткое резюме:

|Что|Как|
|---|---|
|**Зачем нужен свой `BeanPostProcessor`?**|Чтобы автоматически изменять или оборачивать бины без изменения их кода.|
|**Как создать?**|Реализовать `BeanPostProcessor` и переопределить 2 метода.|
|**Как Spring узнает о нём?**|Через `@Component` или регистрацию в конфигурации.|




## @Autowired & @Value



###  `@Autowired`


![[Pasted image 20250604202932.png]]


![[Pasted image 20250604203021.png]]

![[Pasted image 20250604203201.png]]

![[Pasted image 20250604203249.png]]





### Что делает:

`@Autowired` — это аннотация для **автоматического внедрения зависимостей**.

Spring сам найдёт нужный бин по типу и "впихнёт" его в нужное место:

- в поле
    
- в конструктор
    
- в сеттер
    

**Главное:** бин должен быть в контейнере Spring (помечен как `@Component`, `@Service`, `@Repository` или создан в `@Configuration`).

То есть `@Autowired` когда мы например в конструктор передаем какой-то класс и помечаем конструктор этой аннотацией то мы даем понять что этот класс нужно взять из ApplicationContext

---

### Как использовать:

**Внедрение через поле:**
```java
@Component
public class CarService {
    
    @Autowired
    private Engine engine;
}
```



Spring сам найдёт бин `Engine` и подставит сюда.

---

**Внедрение через конструктор:**
```java
@Component
public class CarService {
    
    private final Engine engine;
    
    @Autowired
    public CarService(Engine engine) {
        this.engine = engine;
    }
}
```




Внедрение через сеттер:
```java
@Component
public class CarService {
    
    private Engine engine;
    
    @Autowired
    public void setEngine(Engine engine) {
        this.engine = engine;
    }
}
```



### Нюансы:

- Если несколько бинов одного типа, надо уточнять, какой именно (например, через `@Qualifier`).
    
- Можно поставить `@Autowired(required = false)`, чтобы игнорировать, если бина нет.
    

---


#### Пример:

Сначала создали бин:
```java
@Component("musicPlayerBean")  
public class MusicPlayer {  
    private ClassicalMusic music;  
  
    // IoC  
    @Autowired  
    public MusicPlayer(ClassicalMusic music) {  
        this.music = music;  
    }  
  
    public void playMusic() {  
        System.out.println(music.getSong());  
    }  
}
```

И получаем его:
```java
MusicPlayer musicPlayer = context.getBean("musicPlayerBean", MusicPlayer.class);  
musicPlayer.playMusic();
```



Пример с внедрением двух зависимостей:
```java
@Component("musicPlayerBean")  
public class MusicPlayer {  
    private ClassicalMusic classicalMusic;  
    private MetallMusic metallMusic;  
  
    @Autowired  
    public MusicPlayer(ClassicalMusic classicalMusic, MetallMusic metallMusic) {  
        this.classicalMusic = classicalMusic;  
        this.metallMusic = metallMusic;  
    }  
  
    public void playMusic() {  
        System.out.println("Classical song: " + classicalMusic.getSong());  
        System.out.println("Metall song: " + metallMusic.getSong());  
    }  
}
```




![[Pasted image 20250604235904.png]]


Данная аннотация позволяет избежать ошибки, когда у нас для внедрения зависимости подходят два и более бина


![[Pasted image 20250605000127.png]]



#### Пример:
```java
@Component("musicPlayerBean")  
public class MusicPlayer {  
    private Music music1;  
    private Music music2;  
  
    public MusicPlayer(@Qualifier("classicalMusicBean") Music music1,  
                       @Qualifier("metallMusicBean") Music music2) {  
        this.music1 = music1;  
        this.music2 = music2;  
    }  
  
    public String playMusic() {  
        return "Playing music: " + music1.getSong() + " and " + music2.getSong();  
    }  
}
```






### 2. `@Value`


![[Pasted image 20250605003007.png]]


### Что делает:

`@Value` — аннотация для **вставки значений**:

- из `application.properties`
    
- или прямо из строки
    

Ты её используешь, когда тебе нужно получить:

- строку
    
- число
    
- булево
    
- или что-то ещё простое
    

---

### Как использовать:

**Подставить строку напрямую:**
```java
@Value("Hello World")
private String message;
```



Подставить из настроек:
```java
@Value("${my.custom.value}")
private String customValue;
```


Тогда в `application.properties` должно быть:
```java
my.custom.value=Привет, мир
```



Пример с числом:
```java
@Value("${server.port}")
private int port;
```



### Нюансы:

- `@Value` работает для простых типов (String, int, boolean, и т.п.).
    
- Можно использовать SpEL (`Spring Expression Language`)



### 3. `@Resource`

### Что делает:

`@Resource` — это **аннотация из Java EE** (не Spring, но Spring её поддерживает).  
Она тоже для **внедрения зависимостей**, но работает чуть иначе, чем `@Autowired`:

- В первую очередь ищет **по имени бина** (а не по типу!).
    

---

### Как использовать:

```java
@Component
public class CarService {
    
    @Resource
    private Engine engine;
}
```



### Отличия `@Autowired` и `@Resource`:

|@Autowired|@Resource|
|---|---|
|Ищет по **типу**|Ищет по **имени**, потом по типу|
|Можно с `@Qualifier` указать имя|Можно через `name`|
|Spring-аннотация|Java-аннотация (javax)|

---

## Короткое резюме:

|Аннотация|Для чего нужна|Как работает|
|---|---|---|
|**`@Autowired`**|Внедрение зависимостей|Ищет по **типу**, можно через конструктор, поле, сеттер|
|**`@Value`**|Подставить значение|Из конфига (`application.properties`) или прямо в код|
|**`@Resource`**|Внедрение зависимостей|Ищет по **имени поля**, потом по типу|

---

### Пример реальной практики:

```java
@Component
public class AppConfig {

    @Value("${app.name}")
    private String appName;

    @Autowired
    private UserService userService;

    @Resource(name = "specialEngine")
    private Engine engine;
}
```



## Проблема циклической зависимости

Это когда два бина ссылаются друг на друга.

Решить эту проблему можно несколькими способами:

1. Setter - при помощи сеттеров и аннотации @Autowired
2. Ленивая загрузка - использовать аннотацию `@Lazy` для ленивой загрузки зависимости. Зависимость будет загружена при первом обращении:
```java
public ClassB(@Lazy ClassA a)
```






## ClassPath Sanning



Теперь мы будем внедрять бины не с помощью xml а с помощью аннотаций.
Сначала подключаем необходимое в xml:
```xml
<context:component-scan base-package="java"/>
```



И теперь каждый класс помечаем аннотацией `@Component`, `@Repository` или `@Service`.


Вот примеры:
```java
@Component(value="pool")  
public class ConnectionPool {  
    private final String username;  
    private final Integer poolSize;  
  
  
    public ConnectionPool(@Value("${db.username}") String username,  
                          @Value("${db.pool.size}") Integer poolSize) {  
        this.username = username;  
        this.poolSize = poolSize;  
    }  
  
    @PostConstruct  
    private void init() {  
        System.out.println("Init ConnectionPool");  
    }  
  
  
  
    @PreDestroy  
    private void destroy() {  
        System.out.println("Destroy ConnectionPool");  
    }  
}
```


```java
@Repository  
public class UserRepository {  
    private final ConnectionPool pool;  
    private final List<ConnectionPool> pools;  
    private final Integer poolSize;  
  
    @Autowired  
    private UserRepository(ConnectionPool pool,  
                           List<ConnectionPool> pools,  
                           @Value("${db.pool.size}") Integer poolSize) {  
        this.pool = pool;  
        this.pools = pools;  
        this.poolSize = poolSize;  
    }  
}
```



```java
@Service  
public class UserService {  
    private final UserRepository userRepository;  
  
    public UserService(UserRepository userRepository) {  
        this.userRepository = userRepository;  
    }  
  
}
```



Все поля у нас `final` а объект создается при помощи конструктора





## BeanDefition Readers



![[Pasted image 20250427155444.png]]




![[Pasted image 20250427155530.png]]


### 1. Через **`BeanDefinitionReader`** (слева)

Это "традиционные" способы:

- **`XmlBeanDefinitionReader`** — читает бин-описания из **XML-файлов** (`applicationContext.xml` и подобные).
    
- **`GroovyBeanDefinitionReader`** — читает бин-описания из **Groovy-скриптов**.
    
- **`PropertiesBeanDefinitionReader`** — читает бин-описания из **.properties-файлов**.
    

➔ Старый способ конфигурирования Spring'а. Раньше везде писали XML.

---

### 2. Через **`ClassPathBeanDefinitionScanner`** + **`AnnotatedBeanDefinitionReader`** (в центре)

Это **сканирование аннотаций** в коде:

- **`ClassPathBeanDefinitionScanner`** — пробегает по пакетам проекта и находит классы с аннотациями типа `@Component`, `@Service`, `@Repository`, `@Controller`.
    
- **`AnnotatedBeanDefinitionReader`** — понимает, как создать бин на основе аннотации.
    

И тут ещё используется **`TypeFilter`**, чтобы **фильтровать классы** по признакам (например, найти только классы с нужной аннотацией).

➔ Это способ, который используется сегодня почти всегда — через аннотации.

---

### 3. Через **`ConfigurationClassBeanDefinitionReader`** (справа)

Это **способ через `@Configuration` классы**:

- `@Configuration` — это специальный класс, который сам создаёт бин'ы.
    
- Внутри `@Configuration` мы пишем методы с `@Bean`, и каждый такой метод создаёт бин вручную.
    

Например:
```java
@Configuration
public class AppConfig {

    @Bean
    public Engine engine() {
        return new Engine();
    }
}
```



|Способ|Как работает|
|---|---|
|`XmlBeanDefinitionReader`, `GroovyBeanDefinitionReader`, `PropertiesBeanDefinitionReader`|Чтение конфигураций из файлов|
|`ClassPathBeanDefinitionScanner` + `AnnotatedBeanDefinitionReader`|Автоматический поиск по аннотациям (`@Component`, `@Service` и др.)|
|`ConfigurationClassBeanDefinitionReader`|Чтение классов `@Configuration` и методов `@Bean`|



## TypeFilters



![[Pasted image 20250427162128.png]]










# Java Based Configuration


![[Pasted image 20250605004204.png]]

![[Pasted image 20250605004236.png]]


![[Pasted image 20250605004347.png]]


![[Pasted image 20250605004431.png]]


![[Pasted image 20250605004547.png]]



![[Pasted image 20250605004733.png]]



Пример:
```java
@Component  
@ComponentScan("com.maxim")  
@PropertySource("classpath:musicPlayer.properties")  
public class SpringConfig {  
}
```





Нужен класс `ConfigurationClassBeanDefinitionReader`(Входят аннотации `@Configuration` и `@Bean`)


Нужно в пакете `config` создать класс `ApplicationConfiguration`:
```java
@Configuration    
public class ApplicationConfiguration {  
}
```


Для работы с properties можно использовать аннотацию `@PropertySource`:
```java
@Configuration  
@PropertySource("classpath:application.properties")  // работа с properties
public class ApplicationConfiguration {  
}
```

Тем самым можно не писать properties в xml-файле



Точно также с помощью аннотации можно задать component scan с помощью аннотации `@ComponentScan`:
```java
@Configuration  
@PropertySource("classpath:application.properties")  
@ComponentScan(basePackages = "com.spring")  
public class ApplicationConfiguration {  
}
```



А в методы main для создания applicationContext нужно использовать класс `AnnotationConfigApplicationContext`:
```java
try(AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(ApplicationConfiguration.class)
```




## @Import и @ImportResource


- `@ImportResource` - позволяет импортировать XML-файл
- `@Import` - позволяет импортировать другие configuration-классы





## @Bean

Вот пример создания бина:
```java
@Configuration  
@PropertySource("classpath:application.properties")  
@ComponentScan(basePackages = "com.spring")  
public class ApplicationConfiguration {  
    @Bean  
    public ConnectionPool pool2() {  
        return new ConnectionPool("Mike", 5);  
    }  
}
```








# Event Listeners



Пример EntityEvent:
```java
public class EntityEvent extends EventObject {  
    private final AccessType accessType;  
  
    public EntityEvent(Object entity, AccessType accessType) {  
        super(entity);  
        this.accessType = accessType;  
    }  
    public AccessType getAccessType() {  
        return accessType;  
    }  
}
```


А вот EventListener:
```java
@Component  
public class EntityListener {  
    @EventListener  
    public void acceptEntity(EntityEvent entity) {  
        System.out.println("Entity:" + entity);  
    }  
}
```










# Spring AOP

**AOP(Aspect-Oriented Programming)** - это парадигма программирования, которая позволяет разделять сквозную функциональность (такую как логгирование, транзакции и безопасность) от основной бизнес-логики
Реализуется при помощи паттерна проектирования **Proxy**


![[Pasted image 20250502233248.png]]



**Aspect(Аспекты)** - это модульные классы, которые содержат сквозную функциональность

**Advice** - это выполняемые действия.
Примеры: перед методом, после метода, при выбросе исключения
![[Pasted image 20250502233714.png]]


**Joint Points** - точки в выполнении программы, куда можно вставить Advice.
Пример: вызов метода UserService


**Pointcut(срез)** - это выражения, которые определяют, где именно должен быть применен аспект


![[Pasted image 20250502234029.png]]




Чтобы работал Spring AOP нужно подключить зависимость:
```xml
<!-- https://mvnrepository.com/artifact/org.springframework/spring-aop -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
    <version>6.2.6</version>
</dependency>
```

И также нужно подключить AspectJ Weaver:
```xml
<!-- https://mvnrepository.com/artifact/org.aspectj/aspectjweaver -->
<dependency>
    <groupId>org.aspectj</groupId>
    <artifactId>aspectjweaver</artifactId>
    <version>1.9.24</version>
    <scope>runtime</scope>
</dependency>
```
(для аннотаций)


И в конфигурации указать аннотацию @EnableAspectJAutoProxy:
```java
@Configuration  
@EnableAspectJAutoProxy  
@PropertySource("classpath:application.properties")  
public class MainConfiguration {  
}
```


Вот пример одного аспекта:
```java
@Component  
@Aspect  
public class LoggingAspect {  
  
    @Before("execution(* com.maxim.TaskManager.*(..))")  
    public void logBefore(JoinPoint joinPoint) {  
        System.out.println("Перед вызовом метода: " + joinPoint.getSignature().getName());  
    }  
  
}
```


Этот advice будет срабатывать перед вызовом метода TaskManager


`@Before` - означает что метод будет вызван до вызова определенного метода

`"execution(* com.maxim.TaskManager.*(..))"` - означает вызов метода с любым возвращаемым значением, с любыми аргументами и путь к классу




Добавил аспект `@AfterReturning`:
```java
@Component  
@Aspect  
public class LoggingAspect {  
  
    @Before("execution(* com.maxim.TaskManager.*(..))")  
    public void logBefore(JoinPoint joinPoint) {  
        System.out.println("Перед вызовом метода: " + joinPoint.getSignature().getName());  
    }  
  
  
    @AfterReturning(value = "execution(* com.maxim.TaskManager.*(..))", returning = "result")  
    public void logAfterReturning(  
            JoinPoint joinPoint,  
            Object result) {  
        System.out.println("После возвращения результата: " + joinPoint.getSignature().getName() +  
                " результат: " + result);  
    }  
  
}
```



