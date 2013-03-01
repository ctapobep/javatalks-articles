Even though Maven is a tool that standardizes a lot of things, it has a failed to standardize work with integration tests. Even though it has a phase called `integration-test`, it's not very convenient to use. Here are possible options maven leaves us with:

####Failsafe Plugin
 - Using failsafe plugin. Articles related to this plugin advise to map this plugin execution onto 'integration-test' and 'verify' phases. Thus it will run tests and then check generated reports which would cause build to fail if there were test failures. But this there is a flip side: `integration-test` phase takes places after `test` phase and thus you can't simply run integration tests without running unit tests. This can be overcome by using Maven Profiles and excluding these or those tests by activating a profile. But Maven Profiles tend to overcomplicate any solution, they are too cumbersome and I doubt anyone likes using them. Thus, here is how you may want to configure it:
 ```xml
<plugin>
  <artifactId>maven-failsafe-plugin</artifactId>
  <version>2.13</version>
</plugin>
 ```
 Do not map failsafe goals to any phase. Now, write a test and place it to usual test location, but instead of naming it `XxxTest.java`, use `XxxIT.java` pattern, this one is default for failsafe. Next you can run `mvn test` which runs unit tests as usually, afterwards you can run: `mvn failsafe:integration-test failsafe:verify`. First goal runs integration tests and second fails the build if there were failed tests.

 ####JUnit Categories or TestNG groups
 The idea is to mark some tests as Unit Tests, others - as Integration Tests. In TestNG everything is trivial, after marking tests with groups, use `mvn test -Dgroups=integration-test`. JUnit is tricky because standard surefire plugin doesn't support Categories. Thus:

