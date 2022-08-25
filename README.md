#Liquibase  
Liquibase позволяет описать/хранить и применять/откатывать изменения для БД.
Аналог линейного (без ветвей) GIT для базы.  
https://youtu.be/JTdcd4DYgEI  
https://habr.com/ru/post/460377/  
https://habr.com/ru/post/460907/  

###Доступные опции в spring-boot   
https://docs.liquibase.com/tools-integrations/springboot/springboot.html

###Зависимость в pom.xml
liquibase-core + liquibase-maven-plugin что бы откатывать изменения из мавена  
**liquibase-core БРАТЬ ИЗ РЕПОЗИТОРИЯ, в спрингбуте кривая**

###application.properties
spring.liquibase.change-log=classpath:/db/changelog/changelog-master.xml
если нестандартный путь или файл не yaml

###changelog
Описывает наборы (changesets) изменений в БД.  
Корневой changelog берём их примера  
https://docs.liquibase.com/concepts/changelogs/xml-format.html  
Если используем SQL-изменения желательно указывать для какой СУБД    
```
<preConditions>
    <dbms  type="postgresql" />
</preConditions>
```
https://docs.liquibase.com/concepts/changelogs/preconditions.html

###Установка меток для удобства просмотра/отката
```
<changeSet author="author" id="100">
    <tagDatabase tag="version 1.0.0"/>
</changeSet>
```

###Хранение измениений
Наборы изменений (версионные changelog) складываем в отдельную папку и подключаем в основной как  
`<include file="v-1.0.0/changelog-v-1.0.0.xml" relativeToChangelogFile="true"/>`  
сами изменения в sql файлах, их включаем в версионный changelog через  
`<sqlFile path="create_tables.sql" relativeToChangelogFile="true"/>`  
https://docs.liquibase.com/change-types/sql-file.html

###Откаты изменений
Для изменений структуры LB генерит скрипт отката сам но можем и прописать его сами если неочевидный или сложный.  
Для изменений данных не генерит только если пропишем сами.  
Если надо подавить автогенерацию и сделать измененеия неотменяемыми в changeset ставим пустой `<rollback/>`  

Откат описываем в секции <rollback>  
```
<changeSet author="author" id="210">
  <sqlFile path="up_salary.sql" relativeToChangelogFile="true"/>
  <rollback>
    <sqlFile path="cancel_up_salary.sql" relativeToChangelogFile="true"/>
  </rollback>
</changeSet>
```

###Запуск отката изменений
Открытаться лучше на Tag (изменение с этим тегом УДАЛИТСЯ) но можно ещё и на несколько ченджсетов или на время.
1) Поставить сам LB и тогда из командной строки
liquibase --changeLogFile=<path to changelog file>/<liquibase MASTER changelog file name>.xml --username=<database username> --password=<database password> --classpath=<path to the liquibase installation>/postgresql-42.2.5.jar --url=jdbc:postgresql://<database url>/<database name> rollback <TAG на который откат>
2) **ЛУЧШЕ** через мавен  
   https://www.baeldung.com/liquibase-rollback
* в pom добавим плагин c настройками
```
<plugin>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-maven-plugin</artifactId>
    <version>4.15.0</version>
    <configuration>
        <propertyFile>${project.basedir}/src/main/resources/liquibase.properties</propertyFile>
    </configuration>
</plugin>
```
* Пишем файл liquibase.properties
```
  changeLogFile = db/changelog/changelog-master.xml
  url= jdbc:postgresql://localhost:5432/postgres
  username = postgres
  password = postgres
  driver = org.postgresql.Driver
```
* Выполняем mvn liquibase:rollback -Dliquibase.rollbackTag=<Имя тега>  
Это удобно через идею - где запуск добавить мавен и вставить параметры 
