Original Article: http://articles.javatalks.ru/articles/31

_**Résumé**_: as it appeared Spock's Lifecycle is not always intuitive, this is especially true for `where` block. Even though it's located inside of a test method, its execution happens outside and what's even more peculiar - it's invoked even before `setup()` happens or fields get initialized.

Spock allows you writing [data-driven tests](http://docs.spockframework.org/en/latest/data_driven_testing.html) in a very easy manner using `where`:
```groovy
def "square function"(int a, int square) {
        expect:
            square == a * a
        where:
            a | square
           -1 | 1
            0 | 0
            1 | 1
}
```
But docs keep silence on how this magic is implemented. Spock's Syntax is not pure Groovy, its code gets transformed with [Groovy AST](http://groovy.codehaus.org/Compile-time+Metaprogramming+-+AST+Transformations). Depending on the result of these transformations we may or may not use some features of Spock.

First, `where` is going to be factored out into a separate method within the same test class. But what's really interesting - it will be invoked *before* `setup()`. The whole thing is [here](https://github.com/spockframework/spock/blob/ffa5e066c2b9134a7fd7c6b677b671232319ac75/spock-core/src/main/java/org/spockframework/runtime/ParameterizedSpecRunner.java):
- First [parameters are  initialized](https://github.com/spockframework/spock/blob/ffa5e066c2b9134a7fd7c6b677b671232319ac75/spock-core/src/main/java/org/spockframework/runtime/ParameterizedSpecRunner.java#L40) - I mean those params that you wrote in `where`
- And only then [test methods get invoked](https://github.com/spockframework/spock/blob/ffa5e066c2b9134a7fd7c6b677b671232319ac75/spock-core/src/main/java/org/spockframework/runtime/ParameterizedSpecRunner.java#L42) along with `setup()`

This means that variables that are initialized in `setup()` won't be seen in `where`:

```groovy
class SomeTest extends Specification {
    int a = 2
    int b = 1

    def setup() {
        b = 1
    }
    def someTest(a, b){
      expect:
        b != a
      where:
        a << [a]
        b << [b]
    }
}
```
In this test we'll get `a == b == 0` which demonstrates the point.

As a consequence all the JUnit Listeners (after all Spock is just an extension of JUnit) that observe `beforeTestMethod` will also be triggered after the data is initialized in `where`. This includes `DependencyInjectionTestExecutionListener` from Spring TestContext. Which means that if you want to use `@Autowired` fields which are injected from the context, you won't be able to use them in `where`, what you'll get there is `null`.
