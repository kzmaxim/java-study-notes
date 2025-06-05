# Maven



**Maven** – это специальный “фреймворк” для управления сборкой проектов. Он стандартизирует 3 вещи:

- Описание проекта;
- Сценарии сборки проектов;
- Зависимости между библиотеками.

Есть несколько вариантов организации кода внутри проекта и самый распространенный имеет вид:

![](https://cdn.javarush.com/images/article/1fcd876d-df8f-4ea6-b5de-b671582da78e/512.webp)


- groupId – пакет, к которому принадлежит приложение, с добавлением имени домена;
- artifactId – уникальный строковый ключ (id проекта);
- version – версия проекта.


## Фазы Maven-проекта
https://javarush.com/quests/lectures/questservlets.level01.lecture04

![[Pasted image 20250415210746.png]]


## Сборка проекта

Если ты хочешь скомпилировать проект, то нужно запустить фазу compile. Это можно сделать с помощью командной строки: **mvn compile** или через интерфейс IDEA – кликнув по пункту **compile**.



## Циклы работы

Все команды maven делятся на три группы – lifecycles. Их называют **жизненными циклами**, так как они задают порядок фаз, которые выполняются во время сборки или определенного жизненного цикла, потому что не все действия Maven являются сборкой.

Всего жизненных циклов три:

- **clean;**
- **default;**
- **site.**

## Плагины

Стандартные жизненные циклы можно дополнить функционалом с помощью Maven-плагинов. Плагины позволяют вставлять в стандартный цикл новые шаги (например, распределение на сервер приложений) или расширять существующие шаги.

Плагины в Maven не являются чем-то экстраординарным, наоборот, это самая обычная и часто встречающаяся вещь. Ведь если вы хотите задать какие-нибудь нюансы сборки вашего проекта, то вам нужно указать нужную информацию в pom.xml. И единственный способ это сделать – написать “плагин”.

Так как плагины являются такими же артефактами, как и зависимости, то они описываются практически так же. Вместо раздела dependencies – plugins, вместо dependency – plugin, вместо repositories – pluginRepositories, repository – pluginRepository.


## Maven properties

Часто встречающиеся параметры Maven позволяет вынести в переменные. Это очень полезно, когда нужно чтобы в разных частях pom-файла параметры совпадали. Например, в переменную можно вынести версию Java, версии библиотеки, пути к определенным ресурсам.

Для этого есть специальная секция в `pom.xml – <properties>`, в ней и объявляются переменные. Общий вид переменной такой:
```xml
<имя-переменной>значение</имя-переменной>
```


Пример:
```xml
<properties> 
	<junit.version>5.2</junit.version> 
		<project.artifactId>new-app</project.artifactId><maven.compiler.source>1.13</maven.compiler.source> <maven.compiler.target>1.15</maven.compiler.target>
</properties>
```





# Plugins. Mojo


![[Pasted image 20250517215254.png]]


`Goal` - это обычный Java-класс, который экстендит абстрактный класс `AbstractMojo` и переопределяет единственный метод `execute()`


Пример вызова goal у плагина:
```shell
mvn compiler:help
```

`compiler` - это плагин
`help` - это goal



# JVM arguments


## **VM Options**

1. `-D` - это user arguments, то есть аргументы устанавливаемые пользователем. Они могут быть любые.
Например:
```shell
-Dtest1
-Dfirst=project
```


2. `-X` - зарезервированные JVM options 
Пример:
```shell
-Xms512m   # Минимальный размер heap'а
-Xmx1024m  # Максимальный размер heap'a
```


3. `-XX`  - используется для конфигурирования JIT-компилятора, Garbage-коллектора и тд


## Program arguments

Это массив `args`


Пример:
```shell
arg1
arg2=test
```






# Archetype plugin


`groupId` - это название группы (как правило указывается компания)

`artifactId` - это название проекта в рамках одной группы

`version` - версия проекта. Указывается в таком виде: `major.minor.increment-qualifier` 
  -  `major` - основная версия проекта
  - `minor` - какой-то фикс, небольшой релиз
  - `increment` - это как правило баг-фикс
  - `qualifier`(необязательно) - если указан `SNAPSHOT` то это значит что проект в разработке (не готовая версия)




# POM. Project Object Model


![[Pasted image 20250517223932.png]]





# Dependency Management

Локальный репозиторий находится в папке m2



# Dependency Scope

`Scope` указывается у зависимости и указывает когда зависимость будет применяться


1. `compile`(по умолчанию) -  зависимость нужна для компиляции проекта
2. `provided` - зависимость используется другим сервисом (например Tomcat использует jakarta, и jararta нужно помечать как `provided`)
3. `runtime` - пример это драйвер postgresql, то есть зависимость нужна во время runtime
4. `system` - его не следует использовать, означает что зависимость лежит на компьютере
5. `test` - зависимость нужна во время выполнения тестов







# Lifecycle

**Project Lifecycle** - это набор фаз (phase), которые следуют по очереди друг за другом
**Phase** - это project lifecycle этап, который состоит из целей (goals)


1. Clean
    -  pre-clean
    - **clean**
    - post-clean
2. Default (build)
	- **validate**
	- **compile**
	- **test**
	- **package**
	- **verify**
	- **install**
	- **deploy**
3. Site
	- pre-site
	- **site**
	- post-site
	- site-deploy



## Clean Lifecycle

Эта фаза отвечает за очистку папки `target`

Запускается при помощи команды:
```shell
mvn clean
```



## Default Lifecycle


### Validate

Валидирует проект на корректность


### Compile

Компилирует исходники из папки `src` и кладет их в папку `target/classes` 


### Test

Запускает тесты

Чтобы пропустить тесты при сборке нужно указать `-Dmaven.test.skip=true` :
```shell
mvn test -Dmaven.test.skip=true
```



### Package

Происходит упаковка проекта в какой-то архив. По умолчанию - в jar-архив


Чтобы поменять в какой архив будет упакован проект, нужно указать в тэге `packaging`




### Verify


К этой фазе не привязаны никакие goals

Но как правило к этой фазе привязываются goals который проверяют результаты интеграционных тестов.

И как раз можно поставить специальный плагин который привяжет goal к этой фазе и будет проверят интеграционные тесты.

Это плагин `failsafe plugin`

Чтобы подключить его нужно создать плагин в разделе `build` и в тэге `artifactId` прописать `maven-failsafe-plugin`

А после этого нужно прописать тэг `executions`:
```xml
<executions>
	<execution>
		<goals>
			<goal>integration-test</goal>
			<goal>verify</goal>
		</goals>
	</execution>
</executions>
```

Тогда будут вызваны тесты помеченные `IT*`, `*IT`, `*ITCase`


Получается Unit-тесты запускает плагин `Surefire`, а интеграционные тесты запускает плагин `Failsafe`




### Install

Сохраняет проект в локальный репозиторий




### Deploy


Берет проект из локального репозитория и пушит его в удаленный репозиторий


В качестве удаленного репозитория чаще всего используется `nexus`


![[Pasted image 20250520192712.png]]



#### Как поднять образ nexus на докере

Создал докер-сеть:
```shell
docker network create nexus-net
```

Создал директорию для данных:
```shell
mkdir -p ~/nexus-data
```

Выдал права:
```shell
sudo chown -R 200 ~/nexus-data
```


Запустил nexus через `docker run`:
```shell
docker run -d \
  --name nexus \
  -p 8081:8081 \
  --restart=always \
  -v ~/nexus-data:/nexus-data \
  --network nexus-net \
  sonatype/nexus3
```




Конфигурация pom.xml для отправки на nexus:
```xml
<distributionManagement>  
    <snapshotRepository>  
        <id>nexus</id>  
        <url>http://localhost:8081/repository/maven-snapshots/</url>  
    </snapshotRepository>  
    <repository>  
        <id>nexus</id>  
        <url>http://localhost:8081/repository/maven-releases/</url>  
    </repository>  
</distributionManagement>
```


Далее заходим в файл `~/.m2/settings.xml`
И в теге servers, server указываем id, username и password






## Site Lifecycle


Генерирует отчёты и документацию по проекту

**Jacoco Plugin** - это плагин для генерации отчета на основе тестов





# Multimidule Project


![[Pasted image 20250524174204.png]]



`<dependencyManagement>` - это тэг в котором указывается те зависимости которые есть в рутовом проекте и которые наследуются дочерними модулями
Это нужно чтобы в дочернем модуле не нужно было указывать версию






# Build Profiles


Переопределяются необходимые значения. Это все прописывается в тэге `profiles`



# Executable Jar

Чтобы указать основной класс для запуска нужно в папке `resources` создать папку `META-INF` и там уже создать файл `MANIFEST.MF` и в нем прописать:
```shell
Main-Class: App.java
```




#### А чтобы работали зависимости можно сделать:

1. Создать папку `lib` в target и поместить все зависимости туда
	Для этого нужно создать плагин в .pom:
	![[Pasted image 20250526225437.png]]
	
2. Создать единый jar-файл со всеми зависимостями
	Для этого нужно добавить плагин `maven-assembly-plugin`
	![[Pasted image 20250526225953.png]]
	И потом добавить туда `configuration`:
	![[Pasted image 20250526230142.png]]








# Release Plugin


![[Pasted image 20250526231048.png]]

Нужно подключить плагин `maven-release-plugin`

Потом нужно подключиться к git с помощью тэга `scm`:
![[Pasted image 20250526231641.png]]






