# Пакет java.util.function


## Predicate

Вот как выглядит этот интерфейс:

```java
public interface Predicate<T> {
	boolean test(T t);
}
```

Используется методом `removeIf`
Пример:
```java
al.removeIf(element -> element.lenght()<5);
```



Объединять условия можно при помощи оператора `and`:
```java
Predicate<Student> p1 = student -> student.avgGrade > 7.5;
Predicate<Student> p2 = student -> student.sex == 'm';

info.testStudents(students, p1.and(p2));
```
Так мы объединили два предиката 


Ну и аналогично есть метод `or`:
```java
info.testStudents(students, p1.or(p2));
```


Есть также метод `negate()` - это отрицание:
```java
info.testStudents(students, p1.negate());
```




## Supplier

Переводится как "поставщик"

Имеет такой вид:

```java
public interface Supplier<T> {
	T get();
}
```
То есть он не принимает параметров и возвращает объект типа T



## Consumer

Переводится как "потребитель"

Имеет такой вид:
```java
public interface Consumer<T> {
	void accept(T t);
}
```
То есть он принимает объект типа T и ничего не возвращает

Используется методом `forEach`:
```java
list.forEach(str -> System.out.pritnln(str));
```




## Function

Вот как он выглядит:
```java
public interface Function<T, R> {
	R apply(T t);
}
```
То есть он принимает объект типа T и возвращает объект типа R


 