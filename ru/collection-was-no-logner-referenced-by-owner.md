Original Article: http://articles.javatalks.ru/articles/23

Полная ошибка выглядит так:
```bash
org.hibernate.HibernateException: A collection with cascade="all-delete-orphan" was no longer referenced by the owning entity instance: org.javatalks.training.hibernate.entity.bag.Book.comments`.
```
Собственно ошибка говорит о том, что была у Сущности коллекция, стоял значит у нее `all-delete-orphan`, а тут вместо этой коллекции Сущности подсунули другую. Код, ответственный за такой случай находится в классе `org.hibernate.engine.internal.Collections`:
```java
if ( e != null && e.getStatus() != Status.DELETED && e.getStatus() != Status.GONE ) {
	throw new HibernateException(
			"A collection with cascade=\"all-delete-orphan\" was no longer referenced by the owning entity instance: " +
			loadedPersister.getRole()
	);
}
```
Разработчики вполне намеренно проверяют, если на коллекцию прежний владелец не ссылается (и владелец не был удален), то это проблема. Почему это проблема к сожалению никто не указывает - видимо это перестраховка на случай когда пользователь случайно перезаписал коллекцию. Правильно все-таки делать `remove()`/`removeAll()` на самой коллекции. Два случая когда такое воспроизводится (код на Groovy):

####Коллекция перезаписывается и владелец update'ится
```java
    @Test
    void 'when updating, if orphan collection is dereferenced, exception thrown'() {
        List<Comment> comments = [new Comment(body: "comment1"), new Comment(body: "comment2")]
        Book original = new Book(title: "with comments", comments: comments)
        bookDao.save(original).flushSession()

        try {
            //it's not allowed to replace collection with cascade=orphan
            original.comments = [new Comment(body: 'new')]
            bookDao.saveOrUpdate(original).flushSession()
            assert false
        } catch (HibernateException e) {
            assert e.message.startsWith('A collection with cascade="all-delete-orphan" was no longer referenced by the owning entity instance')
        }
    }
```

####Владелец merge'ится, однако новая коллекция - null

А вот этот случай хитрый. Обычно если мы merge'им детачнутую сущность, то коллекция нормально обновляется - отсутствующие в новой коллекции элементы будут удаляться. Но не тут-то было когда коллекция является null'ом:
```java
    @Test
    void 'when merging, if new collection is null, exception thrown'() {
        List<Comment> comments = [new Comment(body: "a"), new Comment(body: "b")]
        Book original = new Book(title: "with comments", comments: comments)
        bookDao.save(original).flushAndClearSession()

        try {
            bookDao.merge(new Book(id: original.id, comments: null))//exception only due to null
            bookDao.flushSession()
            assert false
        } catch (HibernateException e) {
            assert e.message.startsWith('A collection with cascade="all-delete-orphan" was no longer referenced by the owning entity instance')
        }
    }
```
