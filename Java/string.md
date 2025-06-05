# Устройство класса String
- `charAt(int index)` - возвращает символ, который стоит на index-позиции в строке
- `split()` - разделяет строку на подстроки
- `join()` - склеивает вместе несколько строк
- `getBytes()` - преобразует строку в массив байт
- `replace()` - заменить символы в строке
- `indexOf()` - возвращает начальный индекс подстроки
- `matches(reqex)` - проверяет что строки совпадают с заданным шаблоном
- `substring(begin, enx)` - возвращает подстроку
- `trim()` - удалить все пробелы
- `replaceAll()` - заменяет все вхождения одной строки в другую
- `replaceFirst()` - замена первого вхождения

## StringTokenizer
Класс, задача которого - разделить строку на подстроки
```java
StringTokenizer имя = new StringTokenizer(строка, разделители)
```
Суть в том вторым параметром мы указываем разделители
```java
while(tokenizer.hasMoreTokens()) {
	String token = tokenizer.nextToken();
	System.out.println(token);
}
```


## StringBuilder

Тип String, который можно изменять

Создать StringBuilder на основе существующей строки:
```java
StringBuilder имя = new StringBuilder(строка);
```

Создать пустую изменяемую строку
```java
StringBuilder имя = new StringBuilder();
```

- `append()` - добавить в конец строки
- `insert(idx, obj)` - вставить в строку
- `replace(start, end, str)` - заменить часть строки
- `reverse()` - развернуть строку


`StringBuffer` - тот же StringBuilder но он потоко-защищенный, т.е. одини поток может использовать это объект