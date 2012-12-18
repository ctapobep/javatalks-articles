While on the internet we can find a lot of disputes on what approach XML or Annotations are better to map Hibernate entities, most of arguments are pretty subjective and depend mostly on personal {!!!}. Here     I'd like to describe cons and pros of each approach from practical perspective.
Pros of Annotations:
- They are standard since you can use JPA annotations in most cases and come down to Hibernate-specific ones only when more precise tuning is required which is not declared by JPA.
- It's easier to kick-off. You have a single class and you put annotations there. That's all, no more files.

The cons will be:
- You 'pollute' your classes with annotations. This might not be true if you have a simple mapping, but when it gets complicated you have much more anntations there than anything else.
- You'll have to write HQL queries right inside of classes which is not readable with all this ""+"".

At the same time, cons of XML-based approach:
- You have additional files (well, this is a con only for simple mapping)
- It's not standard, it's not a part of specification which surely would complicate things if you're eager to move to another JPA provider

And finally, here are some pros of XML:
- It leaves your classes clean, especially this is visible when it comes to complex mappings with a lot of details
- You can write your long HQL queries inside hbm.xml files and they would look much better than in Java
- XML is easier for beginners because you have XSD/DTD and all the nice things like autocomplete, you know exactly what's allowed and what's not. In case of annotations you have to know what to put where, and there are times when it's not trivial to google it.
- It's possible to map the same entity to several tables. Example: imagine you have a StoreItem which can have a set of properties. Those properties should have definition: image, weight, color, etc. Those pr    operties should be defined somehow. So you can have a table PROPERTY_DEFINITION that contains available properties along with their default values. Then you have a table with actual values of some Store Items     ITEM_PROPERTY. While creating a Store Item you're copying some properties from PROPERTY_DEFINITION into the ITEM_PROPERTIES. The latter one is referenced by Store Item. So you have 2 identical tables: PROPER    TY_DEFINITION and ITEM_PROPERTIES, but those tables have different logical value. So in Java you can have the same class Property mapped both to PROPERTY_DEFINITION and ITEM_PROPERTIES and thus escape duplica    ting identical classes. It would be impossible with annotations.
- If you share classes between multiple projects in some kind of jars, it's possible that each of your projects can map entities as they want instead of how those entities are mapped inside of those jars. Som    etimes this is a big factor and it leaves flexibility to change those mappings as you wish. Note, that I'm not advocating of having such shared modules, it's just this happens on practice.
- Also sometimes you have XSDs or some other representation you create classes from, they might not be committed and you don't have anything to put annotations on. In case of XML based approach you can keep y    our hbm.xml under your SCM which classes still can be generated during project build. This is of course a pretty rare case because usually you don't want your XML & DB representation be the same classes, ofte    n they differ.

To sum up it's always up to your preferences, you just need keep in mind some potential problems or benefits of other approaches.
