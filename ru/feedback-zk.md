После некоторого использования ZK Framework он оказался хорошим решением для очень простых приложений, однако в реальной жизни проявились его неприятные особенности (здесь я приведу опыт 2012 года, к текущему времени могло что-то поменяться). 

### Отрицательные стороны

**Ужасное community, такое же как и код**. Платную поддержку не использовал и сказать ничего о ней не могу, скажу о бесплатной. Форум у ZK присутствует, и даже ответы какие-то есть. Однако из тех вопросов, которые я там задавал (а их было штук 10) почти ни на один отвечено не было. Это не такая большая проблема если исходники на руках, однако в них такой г-код, что понять как что работает очень сложно. К тому же все построено на событиях и есть некий Event Bus, что еще более усложняет понимание внутреннего исходного кода.

Также мало кто использует этот фреймворк по сравнению с теми же JSF, Spring MVC, поэтому помощи обычно ждать не откуда.

**MVVM sucks**. MVVM (Model-View-ViewModel) - это подход, когда у страниц есть маппинг к определенному объекту и при обновлении состояния Вида обновляется и объект. Ну и наоборот. В демках выглядит все просто прекрасно (собсно поэтому и решился использовать этот MVVM), вот Вид:
```xml
<textbox value="@load(vm.person.name) @save(vm.person.name, before='save')"/>
```Тот сохраняет значение поля в vm.person.name, оттуда же и берет значение для заполнения UI. Все круто пока не сталкиваешься с реальной жизнью, когда UI становится чуть сложней чем элементарный. Есть несколько проблем у этого подхода:

- Он использует состояние в памяти. В отличие от MVC, который обычно просто сохраняет данные на каждый запрос в БД, MVVM как правило хранит все в памяти. От этого вы вскоре достаточно быстро начнете испытывать проблемы, потому как состояния хранить нужно будет все больше.
- В сложных (да даже обычных) UI обычным binding'ом не обойтись - нужно какой-то pre/post processing, нужно обновлять 10 других мест. Да, это все возможно и здесь, только опять же, потому что изменять нужно не одно место (хранилище данных), а много (разные вьюхи, которые в памяти храняться, то же хранилище данных), это становится задачей несколько нетривиальной.

Более детальные негативные отзывы вы можете почитать в блогах .Net разработчиков (которые тоже в свое время пропагандировали этот шаблон).

На самом деле ZK поддерживает как MVVM, так и MVC с MVP, однако больше всего они счас занимаются именно MVVM и его PR'ят. Собственно не ведитесь на удочку.

**ZK позволяет разрабатывать только простой UI**. Вообще ZK себя зарекомендовал как фреймворк для админок. Потому что там особо не надо выпендриваться со сложным и красивым UI. Простые элементы - это конечно хорошо, но даже для админок иногда приходиться сделать что-то более сложное, чем combobox. В такие моменты приходится разбираться не только с JavaScript'ом, но и с тем как писать компоненты для ZK.

**Дружелюбность SEO**. ZK приложения как правило одностраничны и под это заточен фреймворк. Проблема в том, что такие приложения либо не индексируются вовсе, либо индексируются плохо. Особенно учитывая что HTML у ZK выходит до визга сложный. У Zkoss [есть некоторые наработки](http://blog.zkoss.org/index.php/2011/03/17/make-zk-application-work-with-seo/) в этом направлении, однако в их эффективность верится мало. 

**Отсутствие поддержки у IDE**. На данный момент известные мне IDE не поддерживают работу с ZK, вот например [запрос для IntelliJ](http://youtrack.jetbrains.com/issue/IDEA-61007), который уже достаточно давно висит без активных действий. Единственное - т.к. ZK - это XML, можно хоть как XML раскрашивать.

### Положительные стороны

**Никакой компиляции Java->JS**. В отличие от GWT/Vaadin, ZK framework не генерирует никаких JS файлов из Java классов, что не замедляет разработку таких приложений.

**Быстрый UI**. Если посмотреть на тот же Vaadin, то создается впечатление заторможенности. Он достаточно медленно реагирует на мышь. А вот что мне понравилось в ZK - так это быстрое время отклика. 