# Optional

***Optional*** является некоей "обёрткой" для значения
Может содержать null и not-null значения

Метод `isPresent()` проверяет содержится ли значение в Optional

```java
Optional<Integer> o = list.stream().reduce((accumulator, element) -> accumulator*element);

if (o.isPresent()) {
	System.out.println(o.get());
}
else {
	System.out.println("Not present");
}
```


Метод `isPresentOrElse` - позволяет задать логику на случай если значения все таки не существует

```java
min.ifPresentOrElse(
	v -> System.out.println(v)
	() -> System.out.println("Value not found")
);
```
В метод `ifPresentOrElse` передается две функции: первая - обрабатывает случай если значение есть, а вторая - если значения нет


`orElse` - позволяет получить альтернативное значение, если метод который возвращает Optional не получил нужного значения
```java
ArrayList<Integer>  list = new ArratList<>(); // пустой list
Optional<Integer> min = list.stream().min(Integer::compare);
System.out.println(min.orElse(-1)); // -1
```
В данном коде мы ищем минимальное значение в пустом списке и соответственно не находим его, и благодаря методу `orElse` выводится -1

`orElseGet` - позволяет задать функцию, которая будет возвращает значение по умолчанию

`orElseThrow` - позволяет выкинуть исключение если не найдено значение
```java
min.orElseThrow(IllegalStateExcpetion::new);
```