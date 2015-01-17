Original Article: http://articles.javatalks.ru/articles/36

*Resume:* article describes how you configure Java compiler in Ant both in a standalone option and as Antrun Maven Plugin. If configured wrong, you may end up with unnecessary fork processes or errors like this:
```
 Unable to find a javac compiler;
 [ERROR] com.sun.tools.javac.Main is not on the classpath.
 [ERROR] Perhaps JAVA_HOME does not point to the JDK.
 [ERROR] It is currently set to "/Library/Java/JavaVirtualMachines/jdk1.7.0_71.jdk/Contents/Home/jre"
```
As it appears, there are subtleties which may drive you crazy before you start understanding what's going on.

## Which javac should Ant choose?

There are 2 options Ant can use to compile code:
- using classes from classpath
- trigger external javac fork (many ways of configuring this)

### Using javac classes from classpath

When you specify this option `<javac compiler="javac1.6">` it will try to load `com.sun.tools.javac.Main` and if it's not found you'll get: `Unable to find a javac compiler`. Why wouldn't Ant find it? If you run Ant as standalone script, it will add `$JAVA_HOME/lib/tools.jar` to the classpath and the class will be found. But if you wrapped Ant by Maven using antrun plugin, Maven won't load `tools.jar` and will directly call Ant classes. Ant cannot load the required compiler class and thus throws the aforementioned error. In order to escape this, you need to add `tools.jar` explicitly as a dependency to the plugin:
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-antrun-plugin</artifactId>
    <dependencies>
        <dependency>
            <groupId>com.sun</groupId>
            <artifactId>tools</artifactId>
            <version>1.5.0</version>
            <scope>system</scope>
            <systemPath>${env.JAVA_HOME}/lib/tools.jar</systemPath>
        </dependency>
    </dependencies>
</plugin>
```
Note, that potentially some JVM distributions may have this jar in different places. Or may not have it at all (these classes reside in a different jar). Use Maven Profiles with different paths to the dependency set. And then activate those profiles by JDK version or another environment property.

### Change javac using JAVA_HOME

It's possible to change javac by changing what Java is used by Ant itself. And it will work for both fork and non-fork options, just change JAVA_HOME when you start Ant: `JAVA_HOME='/path/to/java/home' ant`.

The rest of options are related only to forked javac.

### Using System javac

System javac is found by Ant in `java.home/../bin` path. Which is based on Java you started Ant with. It's used when Ant is trying to fork an external javac, but the location of javac wasn't specified explicitly. 2 options to do that: `<javac compiler="extjavac">` and `<javac fork="true">`

Since Ant uses `java.home` to locate a system JDK, you may be tempted to try changing the property to point to a different JDK (not the one Ant was started with). I wouldn't recommend it, but it's still worth mentioning. If Ant is invoked directly by another tool (e.g. it was wrapped by Maven or Gradle), it's possible in their code or configuration to change system properties like `java.home`. Note, that it should point to JRE inside of JDK. This is by far the most complicated and subtle way of changing javac in Ant.

### Explicitly specifying javac

You can specify what javac to use (and it again will be a fork): `<javac compiler="extjavac" executable="/path/to/javac">` and `<javac fork="true" executable="/path/to/javac">`. This will find javac itself as well as related JDK (e.g. tools.jar). Note, that if you specify `executable` without `extjavac` or `fork=true`, the option will be ignored and you won't actually change javac.

## To fork or not to fork

In many situations it's vital to restrict resources consumed by processes. CI Server shared between many teams would be one example since you don't want a build to eat all resources and block other builds. In this case a CI engineer would set the memory limits (Heap, PermGen) in build or CI-wide. And while the original Ant process will use these to restrict its appetite, other processes spawned (forked) by the original build will be able to eat more. Ergo, it's better not to use forks if it's possible.

## JAVA_HOME vs. java.home

Many threads about javac task talk about `JAVA_HOME`, I thought it will be appropriate to squeeze in this information here. These variables might be confusing, but they are different things that point to different locations. While `JAVA_HOME` is an environment variable set by user himself (or installer), `java.home` is a system property set by JVM during bootstrap and it usually points to `$JAVA_HOME/jre`. Though it may not. It's not guaranteed because it's determined by several factors. More details can be found in JVM code, see [init_system_properties_values()](http://hg.openjdk.java.net/jdk7/jdk7/hotspot/file/03e1b9fce89d/src/os/linux/vm/os_linux.cpp) function (thanks to [this SO post](http://stackoverflow.com/a/16534705/886697) for pointing that out).
