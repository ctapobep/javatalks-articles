Original article: http://articles.javatalks.ru/articles/32

In every team where I happened to work with [Sonatype Nexus](http://www.sonatype.org/nexus/) I've seen the same misunderstanding on what `LATEST` version of artifacts is. People didn't always understand why they *shouldn't* use it.

Every artifact has a meta file `maven-metadata.xml`, which looks something like this:
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

Now let's imagine you want to download the latest version of the artifact. Of course first links in google will lead you to the REST query:  http://somerepo.org/service/local/artifact/maven/redirect?r=central-proxy&g=groupId&a=artifactId&v=LATEST And maybe, just maybe, it will work for some time. But later at some point Nexus will start returning old artifacts.

This is because when Nexus searches for `LATEST` first it checks whether `maven-metadata.xml` has `<latest>` and chooses it. If this tag is absent (which is usually true at first), then the last item in `<versions>` is used.

This mechanism meets your expectations for some time because at the beginning `maven-metadata.xml` doesn't have `<latest>`. Versions are added to the end of the list as new artifacts get uploaded which means that the latest version is returned. But then one of these happens:
* Someone/something puts `<latest>` to the meta file.
* Developers work in branches and several teams independently upload artifacts with different versions. The last upload is the winner - one of artifacts will be listed in the end of `<versions>` and therefore it will be chosen as `LATEST`.

As it appears [in Nexus LATEST works correctly only for plugins](https://docs.sonatype.com/display/SPRTNXOSS/Nexus+FAQ#NexusFAQ-Q.The"latest"and"release"tagsinmaven-metadata.xmlarenotbeingupdatedafterdeployingartifacts) and this mechanism MUST NOT be used for casual artifacts.

Couple of Q/As:

#### Who updates `maven-metadata.xml`?

Albeit many think that Nexus is the one, it's actually Maven Deploy Plugin who does the trick: it first fetches existing `maven-medatadata.xml` from remote repo, then it updates all the required information and uploads it back to the repo. If the file doesn't exist in the first place, then it gets created and uploaded.

#### How come `<latest>` shows up if Maven doesn't use it?

Maven doesn't actually add this tag. Nexus has its utility mechanisms, one of them is *Rebuild Metadata*. This item can be found in the context menu of the repositories and folders. It removes existing `maven-metadata.xml`, then it iterates over the artifacts and creates a brand new meta file according to what it found. And this is when `<latest>` is added. What for? I wish I knew. I even noticed `<latest>` in cases no one (presumably) ran Rebuild Metadata, thus one may assume there are other events that trigger these updates. Note that during Rebuild the versions are sorted according to the [internal algorithms of Maven](http://sonatype.github.io/sonatype-aether/apidocs/org/sonatype/aether/util/version/GenericVersionScheme.html), these algorithms are then [used by Nexus](http://books.sonatype.com/nexus-book/reference/confignx-sect-managing-tasks.html):
> The ordering is derived by parsing the version string and supports sematic versioning with additional semantics for specific classifiers. Further details can be found in the documentation for the implementing class GenericVersionScheme.

This means that what was LATEST before may not be latest anymore.

#### Why <latest> is not always up-to-date?

Even if you happened to start Rebuild Metadata, this is still a one-time action. Afterwards it's Maven who updates meta files. And it won't do anything with `<latest>` - Maven will simply copy it from the `maven-metadata.xml` originally obtained from Nexus.

#### **Let's sum up**

If you need a mechanism to fetch the latest version of the artifact, you MUST implement it yourself (presumably you'll write a script). Those out-of-box mechanisms Nexus provides will simply stop working at some point.

Also note, that if you think you need the latest versions, you're probably moving in a wrong direction, I haven't seen good workflows which require latests.

**PS**: the links on Sonatype Wiki are now broken, they probably moved their pages or removed them from public access. URLs will be updated when I find new locations.
