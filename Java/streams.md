# Stream

[[lamda|Статья про лямбда-выражения]]


**Stream** - последовательность элементов, поддерживающий последовательные и параллельные операции над ними.

## Методы Stream

- `map` - берет элемент из стрима и сопоставляет ему тот элемент, который мы получаем из лямбда-выражения
Пример создания стрима c `map`

```java
list.stream()
	.map(element -> element.length())
	.collect(Collectors.toList());
```
Методы стримов не меняют коллекцию на которой были применены методы стримов

`Arrays.stream(array)` - преобразовать массив в стрим

```java
Arrays.stream(array)
	.map(element -> 
	(if (element % 3 == 0) {
		element = element / 3;
	}
	return element;)).toArray();
```



- `filter` - фильтрует данные

```java
students.stream()
		.filter(element ->
			element.getAge() > 22 && element.getAvgGrade()< 7.2)
		.collect(Collectors.toList());
```

- `forEach` -  позволяет проходить по всем элементам в стриме, возвращает void

 ```java
 list.stream().forEach(el -> {
	 el*2;
	 System.out.println(el);
 })
```
Просто вывести элементы можно вывести так:
```java
list.stream().forEach(System.out::println);
```
Это называется ***method reference***

***Method reference*** - обозначает, что мы берем каждый элемент нашего стрима и вставляем его в качестве параметра для референс-метода


Стрим можно создать и не из коллекции, а заново, то есть создать новый стрим напрямую:

```java
Stream<String> myStream = Stream.of(str1, str2, str3, str4, str5);
```

И потом можно работать с этим стримом:

```java
myStream.filter(s -> s.length >= 4)
		.collect(Collectors.toList());
```


- `reduce` - после выполнения этого метода в стриме остается один элемент. Удобно для агрегирования

```java
int result = list.stream().reduce((accumulator, element) -> 
	accumulator*element).get();
```

 Как происходит работа метода ***`reduce`*** в данном примере:
 Мы берем переменную `accumulator` и `element`. `accumulator` берет первое значение, а `element` - второе.  В `accumulator` записывается значение умножения этой переменной на `element`, а в сам `element` записывается следующее в стриме значение. И так происходит далее - в `accumulator` записывается результат умножения, а `element` просто берет следующее значение.


 А метод `get()` позволяет получить значение из `Optional`
[[optional|Что такое Optional]]




В `reduce` можно было бы еще указать первым параметров *identity* - это начальное значения для `accumulator`:

```java
int result = list.stream().reduce(1, (accumulator, element) -> accumulator*element);
```
Здесь мы указали что начальным элементов для `accumulator` будет 1

 
Ну и еще один пример для `reduce` - конкатенация всех строк из коллекции:

```java
String result = list.stream().reduce((a, e) -> 
	a + " " + e).get();
System.out.println(result);
```

 

- `sorted` - делает сортировку по заданному лямбда-выражению

Пример
```java
Arrays.stream(arr).sorted().toArray();
```
Здесь мы сделали сортировку массива

Пример с сортировкой студентов (добавляем компаратор):
```java
students.stream()
	.sorted((x, y) -> 
		x.getName().compareTo(y.getName()))
	.collect(Collections.toList());
```


## Method chaining

Это цепочка вызовов методов

Пример 1:
```java
Arrays.stream(arr).filter(e->e%2==1)
	.map(e->{if(e%3==0) {e=e/3;} return e;})
	.reduce((a, e) -> a+e).getAsInt();
```
Сначала фильтруем те элементы которые не делятся на 2, потом те элементы которые делятся на 3 мы делим на 3, а потом складываем эти элементы

Пример 2: 
```java
students.stream().map(element -> {
	element.setName(element.getName().toUpperCase());
	return element;
}).filter(element -> element.getSex() == 'f')
	.sorted((x, y) -> x.getAge() - y.getAge())
	.forEach(element -> System.out.println(element));
```
Здесь мы проходимся по коллекции студентов, сначала преобразуем имена в верхний регистр, потом фильтруем и пропускаем только женщин, потом сортируем по возрасту, ну и в конце выводим все эти элементы


#### Работа метод chaining в stream

`Source(Напр:коллекции, массив)` ---> `Intermediate methods(lazy)` 
---> `Terminal method(eager)`    


![[Pasted image 20250306230740.png]]



`Intermetiate methods` ***не работают до тех пор, пока не будут выведены терминальные методы***

То есть пока не будет вызван терминальный метод, ни один intermediate method не будет вызван




- `concat` - делает конкатенацию стримы, не может быть использован в цепочке
- `distinct` - возвращает стрим уникальных элементов, а сравнивает элементы он с помощью ***equals***
Пример
```java
list.stream().distinct().forEach(System.out::println);
```


- `count` - считает количество элементов в стриме, это terminal method, возвращает *long*
- `peak` - это своего рода forEach но он при этом intermediate method, то есть он возвращает не void, а stream
Пример:
```java
list.stream().distinct().peek(System.out::println).count()
```
Здесь сначала возвращается стрим с уникальными элементами, потом они выводятся, и подсчитываются 



- `flatMap` - используется тогда, когда мы хотим работать не с самими элементами коллекции, а с элементами элементов коллекции
Пример:
```java
list.flatMap(faculty -> faculty.getStudentsInFaculty().stream())
	.forEach(e -> System.out.println(e.getName()));
```
Здесь мы берем факультет из коллекции, затем берем студентов из этого факультета и преобразуем их в стрим, а потом выводим имена этих студентов



- `collect` - преобразует стрим в какую-либо коллекцию 
    -  `groupingBy` - группировка  
Пример
```java
students.stream().map(e -> {
	e.setName(e.getName().toUpperCase());
	return e;
}).collect(Collectors.groupingBy(e -> e.getCourse()));
```
Здесь стрим преобразуется в стрим, который состоит из имен студентов, а потом мы группируем по курсам студентов, то есть создаем мапу по разным курсам которая состоит из имен студентов (ключ - курс, значение - список имен студентов)

   -  `partitioningBy` - разделение по признаку
Пример
```java
students.stream()
		.collect(Collectors.partitioningBy(e -> e.getAvgGrade()>7));
```
Здесь мы разделяем студентов на тех у кого средняя оценка больше 7 и меньше. В результате получается мапа с двумя значениями, где ключи - это true и false (в зависимости от того удовлетворяет условию или нет), а значения - это листы со студентами 


- `findFirst` - терминальный метод, возвращает первый элемент стрима 
- `min` и `max` - находят минимальный и максимальный элементы (можно описывать компаратор)
- `limit` - возвращает стрим, ограничивает количество элементов в стриме
Пример:
```java
list.stream().filter(e->e.getAge()>20).limit(2).forEach(System.out::println);
```

- `skip` - пропускает первые n элементов стрима
```java
list.stream().filter(e->e.getAge()<20).skip(2).forEach(System.out::println);
```


- `mapToInt` - возвращает int stream
Пример:
```java
students.stream()
		.mapToInt(e -> e.getCourse())
		.boxed()   // преобразует int в Integer
		.collect(Collectors.toList())
```

	- `sum` - находит сумму элементов 
	- `average` - находит среднее арифметическое
	- `min` - находит минимум
	- `max` - находит максимум





## Parallel stream


**Parallel stream** - это возможность использовать несколько ядер процессора при выполнении каких-либо операций со stream

### Как сделать стрим параллельным?

1.
```java
list.parallelStream(). ...
```
2.
```java
Stream<T> s = Stream.of(...);
s.parallel(). ...
```


`Использовать параллельные стримы нужно тогда, когда мы работаем с агрегированными функциями, и абсолютно не подходит если наша задача напрямую связана с порядком элементов в стриме`

