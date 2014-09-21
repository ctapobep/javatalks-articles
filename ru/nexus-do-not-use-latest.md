Исходники статьи на [JavaTalks Articles](http://articles.javatalks.ru/articles/22).

Везде, где мне приходилось работать с [Sonatype Nexus](http://www.sonatype.org/nexus/) в какой-то момент возникало недопонимание о том что такое LATEST версия артефакта и почему ее нельзя использовать.

Любой артефакт обладает неким информационным файлом `maven-metadata.xml`, содержимое которого приблизительно такое:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<metadata>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-javadoc-plugin</artifactId>
  <versioning>
    <latest>2.9.1</latest>
    <release>2.9.1</release>
    <versions>
      <version>2.8.1</version>
      <version>2.9</version>
      <version>2.9.1</version>
    </versions>
    <lastUpdated>20130628064237</lastUpdated>
  </versioning>
</metadata>
```

Итак, вы захотели получить последнюю версию артефакта. Вы быстро нагуглите как это можно сделать: http://somerepo.org/service/local/artifact/maven/redirect?r=central-proxy&g=groupId&a=artifactId&v=LATEST И первое время это возможно даже будет работать. Однако в какой-то момент Nexus начнет вам возвращать *старые артефакты*. 

Это происходит из-за алгоритма, который использует Nexus для нахождения LATEST: сначала смотрит если в `maven-metadata.xml` есть `<latest>` и возвращает его. Если тега нет, то вернется последняя версия в списке `<versions>`. 

Изначально механизм этот работает лишь потому, что в `maven-metadata.xml` просто нет `<latest>`. Версии отсортированы в порядке добавления, возвращается последняя. Но в какой-то момент происходит что-то из двух:
* Кто-то таки проставляет `<latest>` в мета-файл.
* Работа идет в ветках и две команды независимо друг от друга заливают артефакты с разными версиями. Кто залил последний - того и последняя версия. Просто-напросто в конец добавляется та версия, которая сейчас деплоится.

Как оказывается [в Nexus LATEST работает правильно только для плагинов](https://docs.sonatype.com/display/SPRTNXOSS/Nexus+FAQ#NexusFAQ-Q.The"latest"and"release"tagsinmaven-metadata.xmlarenotbeingupdatedafterdeployingartifacts) и использовать этот механизм для обычных артефактов нельзя.

Несколько ожидаемых вопросов:

#### Кто обновляет `maven-metadata.xml`?

Этот файл обновляется не Nexus'ом, а Maven Deploy Plugin'ом: сначала он забирает из репозитория существующий 'maven-medatadata.xml', затем обновляет информацию и заливает обратно в репозиторий. Если существующего файла нет, создается новый и заливается в Nexus.

#### Откуда берется `<latest>` если Maven его никак не использует?

Maven этот тег и не проставляет. У Nexus'a есть некие утилитарные механизмы, среди таких - Rebuild Metadata. Этот пункт можно найти в контекстном меню репозиториев и каталогов. Этот механизм удаляет существующую `maven-metadata.xml`, просматривает артефакты и создает новый файл. И в этот момент он создает `<latest>`. Для чего - хрен знает, но создает. Причем я замечал `<latest>` даже в том случае, когда Rebuild Metadata вроде как и не делался, поэтому есть подозрения что это происходит еще по каким-то событиям. Имейте в виду, что сортировка версий происходит исходя из внутренних соображений Nexus'a - скорей всего используются [общие Maven'овские правила](http://mojo.codehaus.org/versions-maven-plugin/version-rules.html). 

#### Почему `<latest>` устаревает?

Даже если Rebuild Metadata и происходит, то это разовое действие. Дальше метадата-файл обновлять все равно будет Maven. А тот просто ничего не делает с `<latest>` - она копируется из того `maven-metadata.xml`, который был загружен из Nexus'a.

#### **Подведем итоги** 

Если вы хотите получить механизм получения последней версии артефакта, этот механизм вы сами и должны будете создать (написать скрипт). Механизм, используемый Nexus'ом по умолчанию в какой-то момент просто перестанет работать.