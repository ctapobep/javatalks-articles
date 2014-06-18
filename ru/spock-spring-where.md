Статья на [javatalks](http://articles.javatalks.ru/articles/edit/27)

*Резюме*: как оказывается у Spock'а порою не совсем интуитивный жизненный цикл, особенно это касается блока `where` который хоть и находится внутри тестового метода, а выполняется не то что до самого метода (что было бы логично), а даже до `setup()` и до инициализации полей класса.

Spock позволяет достаточно удобно строить [data-driven тесты](http://docs.spockframework.org/en/latest/data_driven_testing.html) с помощью своего where:
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
Нигде только не упоминается о том как же в итоге это все реализуется. Синтаксис используемый Spock'ом на самом деле не является чистым Groovy, этот код потом трансформируется с помощью [Groovy AST](http://groovy.codehaus.org/Compile-time+Metaprogramming+-+AST+Transformations). Так вот от того *во что* он трансформируется зависит возможность использования некоторых фич Spock'a.

Так, например, `where` будет вынесено отдельным методом в тот же класс и, что интересно, будет вызван *до* `setup()`. Цепочку можно проследить [тут](https://github.com/spockframework/spock/blob/ffa5e066c2b9134a7fd7c6b677b671232319ac75/spock-core/src/main/java/org/spockframework/runtime/ParameterizedSpecRunner.java):
- Сначала [генерируются параметры](https://github.com/spockframework/spock/blob/ffa5e066c2b9134a7fd7c6b677b671232319ac75/spock-core/src/main/java/org/spockframework/runtime/ParameterizedSpecRunner.java#L40) из `where`
- А затем только [выполняются методы тестовые](https://github.com/spockframework/spock/blob/ffa5e066c2b9134a7fd7c6b677b671232319ac75/spock-core/src/main/java/org/spockframework/runtime/ParameterizedSpecRunner.java#L42) методы наряду с `setup()`

Это значит так же что переменные которые инициализируются в `setup()` не будут видны в `where`:

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
И в итоге мы все равно получим в тесте a == b == 0.

Из этого так же следует, что все слушатели, которые навешаны на `beforeTestMethod` тоже отработают после того как данные будут инициализированы в `where`. К таким слушателям относятся и `DependencyInjectionTestExecutionListener` из Spring TestContext. Т.е. если вы захотите использовать `@Autowired` поля которые инжектятся из контекста, в `where` вы их использовать не сможете, там у вас все равно будут `null`.
