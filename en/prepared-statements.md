_NB: in this article we talk mostly about PreparedStatement, but most of the information is suitable for CallableStatement as well._
Most of us know that JDBC can operate with different kinds of Statements including simple `Statement`, `PreparedStatement` and `CallableStatement`. Let's talk about their purpose and nuances.
First of all it's worth mentioning that Databases when they receive an SQL query they check its syntax, parse it (soft parse), optimize it (hard parse) and create a query plan for it. Query Plan is a DB-level instruction on how exactly it's going to execute the query.
DBs can cache queries they execute, thus if we're doing something like this: `select * from books` and then invoke it several times, then at some point Database will start caching it. In most systems a cache is just a map, which means it has a key and a value. In our case the key is the query (well, its hash) and the value is the Query Plan. When DB receives next identical query, it checks whether there is a Query Plan already prepared waiting to be fetched from the cache. If it's there, then Database won't need to parse it again and we'll get a performance boost (well, it still re-builds the Query Plan in some cases when it wants to optimize the query even more).
It's worth noticing one more time that the key in the cache map is a query. This means that this SQL: `select * from books where id=1` and this one: `select * from books where id=2` are different, they both will be compiled. Thus we can't cache the same query just because it has different parameters! Well, that's exactly the case `PreparedStatement` can cope with. It's possible to rewrite the query to: `select * from books where id=?`. Then each time we want to execute it, Database will take the Query Plan from the cache, the only thing left is just to pass parameters which will replace the question mark. Of course different Databases want to be the best and they work with cache differently, some of them might invalidate cache entries quickly, others will wait longer, the decision may depend on statistics after all.
Now what happens when we use JDBC:
1. We start with `connection.prepareStatement("some query")`, JDBC Driver asks Database to prepare the statement*. DB answers with the identifier of the query (most probably - its hash) and additional data like number of params in the query.
2. Next we invoke `executeQuery()` and JDBC Driver sends the identifier of the query and params. Database finds the query by its ID and simply uses its Query Plan without the need of parsing the query again.

Some of you may notice, that next time we invoke `connection.prepareStatement()` there will be another communication to the Database in order to prepare the query, and then yet another communication for actual query execution. First of all there are 2 extra network requests which is bad from performance point. Second, why the heck do we prepare it again while we already have query ID? Wouldn't it be simpler to reuse the same `PreparedStatement`? Well, that's exactly what happens under the hood!
`Connection#prepareStatement(String sql)` - this is where magic takes place. By that query we pass to the method JDBC Driver checks the object in its internal cache (not a Database cache!) and if it's there, then old `PreparedStatement` is returned. If not, a new object is created**. This is called implicit caching***.
Next stop: `PrepareStatement#close()` - this method doesn't quite stand for its name, it doesn't actually close the statement but rather places it to the internal cache. Physically `PreparedStatment` gets closed only in cases a) if `Connection` is closed b) when cache reaches its max capacity and we need to empty it from old and rarely-used statements c) if cache is switched off d) if cache is not supported by JDBC Driver.

Finally, here are some points related to MySQL (well, most of the stuff will still be common to other Databases):
* SQLs have to be identical (queries with words USERS and users are different!) - that's true for all the Databases****
* PreparedStatements are not always cached the first time they're executed, sometimes you need to query database multiple times.
* Connections to different MySQL Servers, or Connections using different protocols, or even two Connections with different encodings - they all will use different caches.
* Query shouldn't start with spaces (well, to be true I'm not quite sure on this, but I'm tired of reading docs already :) For PostgreSQL this is true).
* Sub-queries and queries with UNION are not cached.
* Queries inside stored procedures are not cached.
* MySQL Server < 5.1.17 doesn't cache the queries, higher versions have their own "ester eggs" which sometimes do not allow caching the queries, so read docs carefully!
* You should set `cachePrepStmts` to true, it's switched off by default. Use connection params like `prepStmtCacheSize` and `prepStmtCacheSqlLimit` for MySQL configuration.

### Security benefits of Prepared Statements?
Besides performance-related features PreparedStatements secures us from SQL Injections. In order to be short, the example will be silly. Let's say we have a forum engine and functionality "Remove User". We specify the username on UI and Submit the form. On the back end we have a code like this: `String query = "delete from users where username=" + username;`. You should remember for your entire life - this is nasty! If a bad guy would pass on UI something like this: `smith ' or 'a'='a` then we'll get such a query: `delete from users where username='smith' or 'a'='a'`. Because `a` always equals to `a` our `where` statement will always be true for all the records in the table. And _all_ of them will be deleted. In order to be safe in this case we would need to escape the string. This means that all the symbols that are meaningful for the Database (like quote symbol)
 should be replaced with some other char sequence. If you do it yourself, it would look like this:
```sql
delete from users where username='smith\' or \'a\'=\'a'
```
Because all the quotes are replaced with \' which means 'interprete the quote as a string, not as a command', we're safe now. But bad guys have a large arsenal, they will always win unless you have a bullet-proof solution. PreparedStatement *is* is this solution. Because in case of PreparedStatements queries look like this: `delete from users where username=?` and we don't construct queries with parameters ourselves: `preparedStatement.setString(1, username)`, we're protected from any kind of SQL Injections - all the escaping is mananged by the database itself.

---

 \* JDBC Drivers that don't support pre-compilation (Prepared Statements) send queries only on the `executeQuery()` step.
 ** Notice that when we create usual `Statement` we don't pass strings which means every time a new object is instantiated.
*** Actually some JDBC Drivers (like Oracle) can cache usual `Statement`s as well. In case of Oracle JDBC Driver you'd need to work with implementation-specific API and it still won't be that effective. That's called Explicit Caching.
**** Of course I didn't look at every single driver, but that's true for 3 of the most popular drivers I've looked into.
