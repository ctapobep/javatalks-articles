Original Article: http://articles.javatalks.ru/articles/33

The full error message looks like this:
```bash
org.hibernate.HibernateException: A collection with cascade="all-delete-orphan" was no longer referenced by the owning entity instance: org.javatalks.training.hibernate.entity.bag.Book.comments`.
```
The error says that Entity had a Collection with `all-delete-orphan` configured. Then instead of changing the original Collection a brand new Collection was set. Here is the code that's responsible for handling such cases in Hibernate (class `org.hibernate.engine.internal.Collections`):
```java
if ( e != null && e.getStatus() != Status.DELETED && e.getStatus() != Status.GONE ) {
	throw new HibernateException(
			"A collection with cascade=\"all-delete-orphan\" was no longer referenced by the owning entity instance: " +
			loadedPersister.getRole()
	);
}
```
The code intentionally checks whether the Collection is referenced by any Entity or not (in case the Owner was removed). Unfortunately code doesn't explain *why* this is a problem - this is probably a way to ensure the user didn't accidentally overwrite the Collection. Anyway the correct way of removing Collections or their items is to use `remove()`/`removeAll()`. Here are 2 cases you can reproduce the error (in Groovy):

#### Collection is overwritten and the Owner gets updated
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

#### Owner gets merged, but new Collection is null

This is a tricky one. Usually if we merge a detached Entity the collection is updated correctly - those elements that are now absent - they get deleted from merged Owner. But if new Collection is null..
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
