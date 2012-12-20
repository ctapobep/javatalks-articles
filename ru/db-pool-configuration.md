[On JavaTalks](http://articles.javatalks.ru/articles/8)

Собственно хочу поделиться и описать конфигурацию DB Pool'a который мы используем в [JTalks](http://jtalks.org):
```xml
    <!--Создаем Ленивый DataSource, который по возможности не будет вытягивать настоящее-->
    <!--соединение к БД из настоящего пула до тех пор пока это правда необходимо. Например, -->
    <!--если Hibernate достает данные из кеша и не делает запрос в БД, нам не нужно-->
    <!--настоящее соединение. Однако если не использовать ленивый DataSource, то соединение-->
    <!--будет вытянуто и транзакция будет начата если метод помечен @Transactional-->
  <bean id="dataSource" class="org.springframework.jdbc.datasource.LazyConnectionDataSourceProxy">
    <property name="targetDataSource">
    <!--Это собственно настоящий DB Pool. Многие говорят что пул, который Hibernate-->
    <!--использует по умолчанию не является серьезным и использование его в production-->
    <!--коде не допустимо. Однако C3P0 как раз не является пулом по умочанию, у Hibernate -->
    <!--есть свой внутренний пул для "поиграться", однако это не C3P0! С другими-->
    <!--же пулами не сравнивал на самом деле, поэтому не могу сказать какой лучше,-->
    <!--C3P0 используется исторически во многих приложениях с Hibernate. -->
      <bean class="com.mchange.v2.c3p0.ComboPooledDataSource" destroy-method="close">
        <property name="driverClass" value="${jdbc.driverClassName}"/>
        <property name="jdbcUrl" value="${jdbc.url}"/>
        <property name="user" value="${JCOMMUNE_DB_USER:root}"/>
        <property name="password" value="${JCOMMUNE_DB_PASS:root}"/>
        <!--Эта опция показывает сколько PreparedStatement'ов должно быть закешировано-->
        <!--на стороне Java, в самом Connection Pool. Имейте в виду что СУБД может -->
        <!--требовать дополнительной настройки для того чтоб эта опция показала себя эффективной-->
        <property name="maxStatements" value="1500"/>
        <!--А это сколько PreparedStatement'ов каждое соединение может кешировать для себя -->
        <property name="maxStatementsPerConnection" value="50"/>
        <!--Вот это самая сложная опция и для того чтоб ее правильно настроить,-->
        <!--нам нужны нагрузочные тесты а также рабочее в PROD приложение-->
        <!--Есть разные стратегии по работе с соединениями которые влияют на оптимальный-->
        <!--размер DB Pool'a, подробней читайте прекрасную книгу Release It!-->
        <property name="maxPoolSize" value="50"/>
        <!--MySQL прибивает соединение если оно не использовалось какое-то время.-->
        <!--По умолчанию это 8 часов. Дабы избежать неприятных исключений нам -->
        <!--нужно бомбардировать MySQL запросами каждый N час (минут)-->
        <property name="idleConnectionTestPeriod" value="3600"/>
        <!--Если мы испытываем большую нагрузку и/или запросы выполняются очень долго,-->
        <!--мы сможем обслуживать 50 пользователей одновременно (размер пула), однако -->
        <!--при увеличении нагрузки, клиенты начнут выстраиваться в очередь и просто ждать-->
        <!--заблокированные в synchronized методах внутри DB Pool'a. Дабы мы не оказались-->
        <!--в ситуации, когда мы заблокированы надолго и приложение совсем не отвечает, -->
        <!--через 10 секунд простаивания в очереди выбрасывается исключение и сервер разгружается-->
        <!--от лишних запросов. Да, это исключение и это неприятно, однако лучше отвалится -->
        <!--несколько клиентов (особенно если это DDoS'еры поганые) нежели сайт будет в полной отключке.-->
        <property name="checkoutTimeout" value="10000"/>
      </bean>
    </property>
  </bean>
```

---
* [Подробрей о кешировании PreparedStatement](http://articles.javatalks.ru/articles/2)
* Если вы получили подобное исключение, значит вам поможет `iddleConnectionTestPeriod` описанный выше:

```
com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: The last packet successfully received from the server was 64,129,968 milliseconds ago.  The last packet sent successfully to the server was 64,129,968 milliseconds ago. is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.
    at com.mysql.jdbc.Util.handleNewInstance(Util.java:411)
    at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:1116)
    at com.mysql.jdbc.MysqlIO.send(MysqlIO.java:3851)
```