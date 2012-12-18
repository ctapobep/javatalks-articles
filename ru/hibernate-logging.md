[On JavaTalks](http://articles.javatalks.ru/articles/5)

Если вы работаете с Hibernate, вам скорей всего понадобится видеть, то ли для оптимизации, то ли для дебага какие запросы и зачем он генерирует. Следующие свойства следует описать в _hibernate.cfg.xml_ ну или передать во время конфигурирования `SessionFactory`. 

Самое важное, это возможность увидеть SQL генерируемый Хибом: 
```xml
<property name="show_sql">true</property>
``` 
Однако запросы появятся только в консоли. Дабы по-настоящему логировать SQL запросы, нужно указать следующий логер в log4j: `org.hibernate.SQL=TRACE`. Обратите внимание на заглавные буквы SQL.

Идем дальше. С этим параметром Хиб будет выводить комментарии в лог о том, зачем тот или иной запрос выполняется - загрузка ли какой-то сущности, удаление ли, просто синхронизация и т.п.:
```xml
<property name="use_sql_comments">true</property>
```

А следующая опция будет форматировать выводимый запрос, иначе запрос будет в одну строку:
```xml
<property name="format_sql">true</property>
```

А такой логер нужно настроить чтоб Hibernate логировал значения передаваемые в `PreparedStatement` вместо `?` и `:name`
```properties 
log4j.logger.org.hibernate.type = TRACE
```
А вот так будет выглядеть файл конфигурации для logback (groovy-версия):
```groovy
import ch.qos.logback.classic.encoder.PatternLayoutEncoder
import ch.qos.logback.core.ConsoleAppender

import static ch.qos.logback.classic.Level.*

appender("consoleAppender", ConsoleAppender) {
    encoder(PatternLayoutEncoder) {
        pattern = "[%d{HH:mm:ss.SSS}] [%thread] [%-5level] [%logger{36}] - %msg%n"
    }
}

/**
 * This would allow us to see parameters passed into prepared statements
 */
logger("org.hibernate.type.descriptor.sql.BasicBinder", TRACE)
/**
 * We don't want to include the whole org.hibernate.type package to get rid of noise, 
 * thus we need to include necessary classes explicitly.
 */
logger("org.hibernate.type.EnumType", TRACE)
/**
 * Shows executed SQL statements. This one is better than show_sql because the latter can log only to console. Note,
 * capitalized SQL letters, it's important.
 */
logger("org.hibernate.SQL", TRACE)
root(INFO, ["consoleAppender"])
```

### DB/Driver Logging
Однако и это не все. Иногда Hibernate не в состоянии логировать запрос как есть, потому как он генерируется самим драйвером, так например происходит во время [Batch операций](http://docs.oracle.com/javase/1.3/docs/guide/jdbc/spec2/jdbc2.1.frame6.html). Для того, чтоб увидеть такие запросы по-настоящему (Hibernate их будет выводить как и другие запросы), нужно смотреть на логи БД или JDBC Driver'a. Рассмотрим пример MySQL. Увидеть запросы можно либо заглянув в `/var/log/mysql/mysql.log` (это может зависить от версии и настроек MySQL Server'a), либо указав в строке соединения параметры логирования:
```properties
jdbc.url=jdbc:mysql://localhost/hib_training?characterEncoding=UTF-8\
  &rewriteBatchedStatements=true\
  &logger=com.mysql.jdbc.log.StandardLogger\
  &profileSQL=true
```
Стандарный логер (он будет использоваться по умолчанию) может вас не удовлетворить потому как он будет выводить сообщения не синхронно с самим Hibernate. По этим или другим причинам вы можете настроить `com.mysql.jdbc.log.Slf4JLogger` либо `com.mysql.jdbc.log.Jdk14Logger`.
