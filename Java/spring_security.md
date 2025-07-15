

![[Pasted image 20250713153218.png]]



![[Pasted image 20250713153249.png]]



![[Pasted image 20250713153420.png]]



![[Pasted image 20250713153604.png]]








![[Pasted image 20250713153803.png]]




![[Pasted image 20250713154151.png]]




![[Pasted image 20250713154413.png]]









# Сессии


![[Pasted image 20250713155217.png]]




![[Pasted image 20250713155442.png]]



![[Pasted image 20250713155457.png]]







# Cookies




![[Pasted image 20250713162623.png]]




![[Pasted image 20250713162703.png]]



![[Pasted image 20250713162940.png]]



![[Pasted image 20250713163130.png]]









# Первое приложение с использованием Spring Security




При запуске приложения Spring Security будет требовать логин и пароль. Он будет считать что доступ ко всем страницам кроме `/login` закрыт.










# Аутентификация в Spring Security


## 📘 Общая суть

Spring Security отвечает за:

- ✅ Аутентификацию (вход по логину/паролю)
    
- ✅ Авторизацию (ограничение доступа к URL)

Чтобы использовать **свою сущность пользователя (`Person`)**, нужно сделать:

1. Обёртку `PersonDetails`, которая реализует `UserDetails`
2. Сервис `PersonDetailsService`, который ищет пользователя в БД
3. Настроить `AuthenticationProvider` (через `AuthProviderImpl`)
4. Настроить `SecurityConfig` (через `SecurityFilterChain` — без устаревших классов)



## 📦 Структура проекта

```arduino
├── config/
│   └── SecurityConfig.java
├── security/
│   ├── PersonDetails.java
│   ├── PersonDetailsService.java
│   └── AuthProviderImpl.java
├── model/
│   └── Person.java
├── repository/
│   └── PeopleRepository.java
```



## 1. ✳️ Сущность `Person`

```java
@Getter  
@Setter  
@NoArgsConstructor  
@Entity  
@Table(name="person")  
public class Person {  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private int id;  
  
    private String username;  
  
    @Column(name="year_of_birth")  
    private int yearOfBirth;  
  
    private String password;  
  
    public Person(String username, int yearOfBirth) {  
        this.username = username;  
        this.yearOfBirth = yearOfBirth;  
    }  
  
    @Override  
    public String toString() {  
        return "Person{" +  
                "id=" + id +  
                ", username='" + username + '\'' +  
                ", yearOfBirth=" + yearOfBirth +  
                ", password='" + password + '\'' +  
                '}';  
    }  
}
```



## 2. 🧍 Обёртка `PersonDetails`


```java
@RequiredArgsConstructor  
public class PersonDetails implements UserDetails {  
  
    private final Person person;  
  
    @Override  
    public Collection<? extends GrantedAuthority> getAuthorities() {  
        return List.of();  
    }  
  
    @Override  
    public String getPassword() {  
        return this.person.getPassword();  
    }  
  
    @Override  
    public String getUsername() {  
        return this.person.getUsername();  
    }  
  
    @Override  
    public boolean isAccountNonExpired() {  
        return true;  
    }  
  
    @Override  
    public boolean isAccountNonLocked() {  
        return true;  
    }  
  
    @Override  
    public boolean isCredentialsNonExpired() {  
        return true;  
    }  
  
    @Override  
    public boolean isEnabled() {  
        return true;  
    }  
}
```



## 3. 🔍 Сервис `PersonDetailsService`

```java
@Service  
@RequiredArgsConstructor  
public class PersonDetailsService implements UserDetailsService {  
    private final PeopleRepository peopleRepository;  
  
  
    @Override  
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {  
        Optional<Person> person = peopleRepository.findByUsername(username);  
  
        if (person.isEmpty()) {  
            throw new UsernameNotFoundException(username);  
        }  
        return new PersonDetails(person.get());  
    }  
}
```



## 4. 🔐 Кастомный `AuthenticationProvider`

```java
@Component  
@RequiredArgsConstructor  
public class AuthProviderImpl implements AuthenticationProvider {  
  
    private final PersonDetailsService personDetailsService;  
  
    @Override  
    public Authentication authenticate(Authentication authentication) throws AuthenticationException {  
        String username = authentication.getName();  
  
        UserDetails personDetails = personDetailsService.loadUserByUsername(username);  
  
        String password = authentication.getCredentials().toString(); // пароль в виде строки  
  
        if (!password.equals(personDetails.getPassword())) {  
            throw new BadCredentialsException("Incorrect password");  
        }  
        return new UsernamePasswordAuthenticationToken(personDetails, password, personDetails.getAuthorities());  
    }  
  
    @Override  
    public boolean supports(Class<?> authentication) {  
        return true;  
    }  
}
```



## 5. ⚙️ Конфигурация `SecurityConfig`

```java
@Configuration  
@EnableWebSecurity  
@RequiredArgsConstructor  
public class SecurityConfig {  
  
    private final AuthProviderImpl authProvider;  
  
    @Bean  
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {  
        http  
                .authorizeHttpRequests(auth -> auth  
                        .requestMatchers("/register", "/error").permitAll()  
                        .anyRequest().authenticated()  
                )  
                .formLogin(Customizer.withDefaults()) // ✅ встроенная форма логина  
                .logout(logout -> logout  
                        .logoutUrl("/logout")  
                        .logoutSuccessUrl("/login")  
                );  
  
        return http.build();  
    }  
  
    @Bean  
    public AuthenticationManager authenticationManager(HttpSecurity http) throws Exception {  
        return http  
                .getSharedObject(AuthenticationManagerBuilder.class)  
                .authenticationProvider(authProvider)  
                .build();  
    }  
  
    @Bean  
    public PasswordEncoder passwordEncoder() {  
        return new BCryptPasswordEncoder();  
    }  
}
```


---

## 📥 Как получить текущего пользователя


```java
Authentication auth = SecurityContextHolder.getContext().getAuthentication();
PersonDetails personDetails = (PersonDetails) auth.getPrincipal();
Person person = personDetails.getPerson();
```



## ✅ Резюме: шаг за шагом

| Шаг | Что делаем                         | Описание                                              |
| --- | ---------------------------------- | ----------------------------------------------------- |
| 1   | Создаём сущность `Person`          | Добавляем поля `username`, `password`                 |
| 2   | Реализуем `PersonDetails`          | Имплементация интерфейса `UserDetails`                |
| 3   | Создаём `PersonDetailsService`     | Реализация метода `loadUserByUsername()`              |
| 4   | Создаём `AuthProviderImpl`         | Кастомная проверка логина и пароля                    |
| 5   | Настраиваем `SecurityConfig`       | Через `SecurityFilterChain` + `AuthenticationManager` |
| 6   | Используем `SecurityContextHolder` | Получаем текущего авторизованного пользователя        |




## 🔁 Что поменялось по сравнению со старым API?


|Было (устарело)|Стало (новое)|
|---|---|
|`WebSecurityConfigurerAdapter`|❌ Удалён|
|`configure(HttpSecurity)`|заменён на `SecurityFilterChain` с бинами|
|`formLogin().loginPage(...)`|⚠️ Всё ещё работает, но можно `Customizer.withDefaults()`|
|`PasswordEncoderFactories.createDelegatingPasswordEncoder()`|лучше использовать `BCryptPasswordEncoder()` напрямую|
|`@Bean AuthenticationManager` вручную|теперь создаётся через `http.getSharedObject(...)`|


![[Pasted image 20250713220740.png]]





![[Pasted image 20250713221031.png]]




![[Pasted image 20250713221350.png]]




![[Pasted image 20250713221546.png]]





Немного меняем конфигурацию:
```java
@Configuration  
@EnableWebSecurity  
@RequiredArgsConstructor  
public class SecurityConfig {  
  
    private final PersonDetailsService personDetailsService;  
  
    @Bean  
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {  
        http  
                .authorizeHttpRequests(auth -> auth  
                        .requestMatchers("/register", "/error").permitAll()  
                        .anyRequest().authenticated()  
                )  
                .formLogin(Customizer.withDefaults()) // ✅ встроенная форма логина  
                .logout(logout -> logout  
                        .logoutUrl("/logout")  
                        .logoutSuccessUrl("/login")  
                );  
  
        return http.build();  
    }  
  
    @Bean  
    public AuthenticationManager authenticationManager(HttpSecurity http) throws Exception {  
        AuthenticationManagerBuilder authBuilder = http.getSharedObject(AuthenticationManagerBuilder.class);  
        authBuilder.authenticationProvider(daoAuthenticationProvider());  
        return authBuilder.build();  
    }  
  
    @Bean  
    public DaoAuthenticationProvider daoAuthenticationProvider() {  
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();  
        provider.setUserDetailsService(personDetailsService);  
        provider.setPasswordEncoder(passwordEncoder());  
        return provider;  
    }  
  
  
    @Bean  
    public PasswordEncoder passwordEncoder() {  
        return new BCryptPasswordEncoder();  
    }  
}
```









# Кастомная страница авторизации



HTML страница авторизации:
```html
<!DOCTYPE html>  
<html lang="ru" xmlns:th="http://www.thymeleaf.org">  
<head>  
    <meta charset="UTF-8">  
    <title>Login</title>  
</head>  
<body>  
  
<form action="/process_login" method="post" name="f">  
    <label for="username">Введите имя пользователя:</label>  
    <input type="text" name="username" id="username" placeholder="Username">  
    <br>  
    <label for="password">Введите пароль:</label>  
    <input type="password" name="password" id="password" placeholder="Password">  
    <br>  
    <input type="submit" value="Login">  
  
    <div th:if="${param.error}" style="color:red;" >  
        Неправильное имя или пароль  
    </div>  
</form>  
  
  
</body>  
</html>
```



И потом в `SecureConfig` нужно добавить вместо дефолтной страницы:
```java
.formLogin( form -> form  
        .loginPage("/auth/login")  
        .loginProcessingUrl("/process_login")  
        .defaultSuccessUrl("/hello")  
        .failureUrl("/auth/login?error")  
        .permitAll()  
)
```










# Процесс регистрации в Spring Security



Для начала нужно реализовать метод контроллера который будет показывать правильное представление:
```java
@GetMapping("/registration")  
public String registrationPage(@ModelAttribute("person") Person person) {  
    return "auth/registration";  
}
```


И потом создать само представление с формой регистрации:
```html
<!doctype html>  
<html lang="en" xmlns:th="http://www.thymeleaf.org">  
<head>  
    <meta charset="UTF-8">  
    <meta name="viewport" content="width=device-width, initial-scale=1">  
    <title>Registration</title>  
</head>  
<body>  
<form th:action="@{/auth/registration}" th:object="${person}" th:method="POST">  
    <label for="username">Введите имя пользователя:</label>  
    <input type="text" th:field="*{username}" id="username">  
    <div style="color:red" th:if="${#fields.hasErrors('username')}" th:errors="*{username}">Username error</div>  
    <br>  
    <label for="yearOdBirth">Введите год рождения:</label>  
    <input type="text" th:field="*{yearOfBirth}" id="yearOdBirth">  
    <div style="color:red" th:if="${#fields.hasErrors('yearOfBirth')}" th:errors="*{yearOfBirth}">Year error</div>  
    <br>  
    <label for="password">Введите пароль:</label>  
    <input type="password" th:field="*{password}" id="password">  
    <div style="color:red" th:if="${#fields.hasErrors('password')}" th:errors="*{password}">Password error</div>  
  
    <input type="submit" value="Sign up!">  
</form>  
</body>  
</html>
```


После этого реализуем метод контроллера который будет обрабатывать эту форму:
```java
@PostMapping("/registration")  
public String performRegistration(@ModelAttribute("person") @Valid Person person, BindingResult bindingResult) {  
    personValidator.validate(person, bindingResult);  
    if (bindingResult.hasErrors()) {  
        return "auth/registration";  
    }  
  
    registrationService.register(person);  
  
    return "redirect:/auth/login";  
}
```


Реализуем валидатор:
```java
@Component  
@RequiredArgsConstructor  
public class PersonValidator implements Validator {  
  
    private final PeopleService peopleService;  
  
    @Override  
    public boolean supports(Class<?> clazz) {  
        return Person.class.equals(clazz);  
    }  
  
    @Override  
    public void validate(Object target, Errors errors) {  
        Person person = (Person) target;  
        Optional<Person> personFromBD = peopleService.findByUsername(person.getUsername());  
        if (personFromBD.isPresent() && personFromBD.get().getId() != person.getId()) {  
            errors.rejectValue("username", "", "Человек с таким именем уже существует!");  
        }  
    }  
}
```


И реализуем `RegistrationService`:
```java
@Service  
@RequiredArgsConstructor  
public class RegistrationService {  
  
    private final PeopleRepository peopleRepository;  
  
    @Transactional  
    public void register(Person person) {  
        peopleRepository.save(person);  
    }  
}
```









# Процесс разлогинивания (logout)



Чтобы сделать механизм ралогинивания (выхода) достаточно прописать этот адрес в `SpringConfig`:
```java
.logout(logout -> logout  
        .logoutUrl("/logout")  
        .logoutSuccessUrl("/auth/login")  
);
```


И потом просто создать кнопку/ссылку на этот адрес:
```html
<a href="/logout">Logout</a>
```

Spring Security сам удалит куки, пользователя из сессии








# Шифрование



![[Pasted image 20250714161544.png]]



![[Pasted image 20250714161640.png]]




![[Pasted image 20250714161856.png]]




![[Pasted image 20250714162006.png]]



![[Pasted image 20250714162140.png]]




![[Pasted image 20250714162223.png]]




![[Pasted image 20250714162326.png]]



![[Pasted image 20250714162438.png]]




Для начала нужно добавить bcrypt в `SecurityConfig`:
```java
@Bean  
public PasswordEncoder passwordEncoder() {  
    return new BCryptPasswordEncoder();  
}
```


И теперь добавляем этот бин в `RegistrationService`:
```java
@Service  
@RequiredArgsConstructor  
public class RegistrationService {  
  
    private final PeopleRepository peopleRepository;  
    private final PasswordEncoder passwordEncoder;  
  
    @Transactional  
    public void register(Person person) {  
        person.setPassword(passwordEncoder.encode(person.getPassword()));  
  
        peopleRepository.save(person);  
    }  
}
```


При помощи `person.setPassword(passwordEncoder.encode(person.getPassword()));` мы зашифровываем пароль.

Теперь в БД пароль выглядит так:

![[Pasted image 20250714163558.png]]









# CSRF



![[Pasted image 20250714182112.png]]



![[Pasted image 20250714182833.png]]



![[Pasted image 20250714182854.png]]




![[Pasted image 20250714183022.png]]




![[Pasted image 20250714183112.png]]





Чтобы защититься от CSRF нужно вставить скрытое поле с токеном в форму авторизации:
```html
<input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}"/>
```


И также нужно поменять кнопку разлогинивания: нужно чтобы она отправляла POST запрос
```html
<form th:action="@{/logout}" th:method="post">  
    <input type="submit" value="Logout"/>  
</form>
```










# Авторизация в Spring Security



![[Pasted image 20250714184127.png]]




![[Pasted image 20250714184415.png]]




Для авторизации нужно настроить `SecurityConfig`:
```java
.authorizeHttpRequests(auth -> auth  
        .requestMatchers("/admin").hasRole("ADMIN")  // для админа
        .requestMatchers("/auth/login","/auth/registration", "/auth/error").permitAll()  
        .anyRequest().hasAnyRole("ADMIN","USER")  // для админа и юзера
)
```


Ну и подправить `RegistrationService` чтобы он выдавал роль юзер зарегистрированным пользователям: 
```java
@Transactional  
public void register(Person person) {  
    person.setPassword(passwordEncoder.encode(person.getPassword()));  
    person.setRole("ROLE_USER");  
  
    peopleRepository.save(person);  
}
```








# OAuth 2.0



![[Pasted image 20250715123511.png]]




![[Pasted image 20250715123633.png]]




![[Pasted image 20250715123747.png]]



