Original Article at JavaTalks: http://articles.javatalks.ru/articles/37

*Resume:* when we're new to a tool, we usually start using this tool in a wrong way. But experience & time changes the way we think of it, we start using it in a different way. Here I'd like to show what stages I personally went through while writing System & Component tests. Many of others go through similar steps, maybe this post will help speed up their personal evolution.

The article will be useful to both Developers (Devs) and Automation Test Engineers (AQA). Devs often write component tests (which they call integration tests, but I hate this term for its vagueness) and AQA usually concentrate on System Functional Tests written e.g. on Selenium WD (I will use it as a reference implementation here). I'd like to start from System Tests because if Devs start writing them, they most probably go through the same problems.

Find the stage you're currently at and proceed with it to get an understanding of what you can expect next. If you think that the last stage is not final, please let me know.

# Homo habilis: Messing test cases & tools

We started exploring the tools like Selenium and our test cases are automated like this:
```
public void clickSignUpLinkTest(String appURL) {
  driver.get(appURL);
  driver.findElement(By.xpath("//a[@href='/jcommune/user/new']")).click();
  driver.findElement(By.id("password")).clear();
  driver.findElement(By.id("passwordConfirm")).clear();
  driver.findElement(By.id("password")).sendKeys(acceptedPassword);
  driver.findElement(By.id("passwordConfirm")).sendKeys(acceptedPassword);
  driver.findElement(By.xpath("//button[@type='submit']")).click();
  assertEquals(driver.getCurrentUrl(), appURL);
}
```
People usually don't stay on this stage for more than several months because it clearly sucks.

*Pros:*
- Fast to write and thus might be a good starting point which afterwards gets refactored into something more solid.
- Fast to write and thus can be more cost-effective if the functionality under test is experimental and the odds of refactoring are high.

*Cons:*
- Not always clear what test case does since it's mixed with low-level logic of Selenium
- If technical part changes (e.g. selectors), test case must be changed even though it didn't change per se
- Test cases may become very large

# Homo erectus: Using Page Objects

We started to understand that selectors should move out into separate objects to get rid of the mess we've written so far. This is where most of AQA get stuck! Now you may start worrying since we're far from the end :) At this stage we have a Page class:
```
public class SignInPage {
  @FindBy(id = "j_username")
  WebElement usernameField;

  public WebElement getUsernameField() { return usernameField; }
}
```

And we use it in the test cases:
```
public void usernameValidationTest(String username) {
  signInPage.getUsernameField().sendKeys(username);
  signInPage.getSubmitButton().click();
  assertExistBySelector(driver, signInPage.errorMessageSel);
}
```
Now test cases are not blotted with stuff like selectors. Some actions like `click()` may also move to Page Object.

*Pros:*
- It's more clear what the test case does step by step
- If selector changes, we don't change the test case itself

*Cons:*
- Test case is still prone to changes because it's page-oriented. What if previously we were moving from PageA to PageB, but after a change we started to move from PageC to PageB? We would need to change both test case and Page Object while test case that checks the validity of some field on page didn't really change.
- We would need different test cases for actions achieved through different paths. Imagine that your pages are adaptive to screen sizes: on large screens your elements differ from those shown on small screens. In page-oriented test cases you'll have to write different tests for different devices while the case doesn't differ.
- The test cases can still be pretty large depending on the complexity of the workflow. Dependency is linear.

# Homo antecessor: Business-Oriented Test Cases

Since we figured out what to improve in Page Oriented Test Cases, let's see how this can be refactored. We would still need Page Objects:
```
public class SignInPage {
  @FindBy(id = "j_username")
  WebElement usernameField;

  public WebElement getUsernameField() { return usernameField; }
}
```

But now in Test Cases we use business operations that don't care about pages:
```
public void usernameAsSpace_shouldFail() throws Exception {
    UserForRegistration user = UserForRegistration.withUsername(" ");
    Users.signUp(user);
}
```

And we've created another layer that implements business operations:
```
public static User signUp(UserForRegistration userForRegistration) throws ValidationException {
    mainPage.logOutIfLoggedIn(driver);
    openAndFillSignUpDialog(userForRegistration);
    checkFormValidation(signUpPage.getErrorFormElements());
    signUpPage.closeRegistrationWasSuccessfulDialog();
    return userForRegistration;
}
```

*Pros:*
- Test cases are very compact - their complexity doesn't quite depend on the complexity of the workflow since it's hidden behind the business layer.
- It's very easy to read such cases even for not-quite-technical people.
- Test cases are fully unbound from pages and technical details. No matter what changes, if Sign Up operation still exists in the app, test case won't change unless the rules of the Sign Up functionality change.

*Cons:* at this point you feel like everything is done in the best possible way - cases are very concise and readable, who knows, maybe they are finally stable. But most probably you still have problems (which could've been present on previous stages of your evolution):
- Test runs take too long. At [JTalks](http://jtalks.org) after we optimized tests, split some of them to run in headless mode (HtmlUnit) they still took about 5 hours to finish. Usually it's not a big deal because tasks are being done continuously and QA always have tasks if Devs work fast enough. But sometimes it becomes a real drag.
- Because of the large number of tests in a single run (we had 100 cases in 4 browsers + 300 headless tests = 700 tests per one run) they will still fail no matter what. At [JTalks](http://jtalks.org) this wasn't often, we could have dozens of green runs, but then out of blue one test would fail. Before we split them into headless & browser tests they were failing much more often, but that still would happen even after the split.

As I said - these problems show up no matter on what level of evolution you are, but then we finally are coming closer to nirvana.

# Homo sapiens: Writing Component Tests

Do you remember those Test Pyramids where you have Unit Tests at the bottom, then Component Tests and only after that System Tests? They do make sense. I don't know why so many people forget about them and write only Unit & System Tests. So here is the bottom line:
- There should be a small number of System Tests.
- Profit!

I haven't implemented Component Tests in web apps yet, but this is currently in progress and soon I'll know this for sure. Here is how we do it:
- System Tests check only UI and no business logic. Let's say you have a Sign In form and there are a lot of validation rules. In System Tests we only need to check situations that differ on UI. E.g. Happy Path and 1 validation error. We don't need to test several Happy Paths like Lower Boundary, Upper Boundary, Special Symbols. One successful case and one negative case should be enough (well, if there are differences in UI for different negative cases, then it might make sense to test them as well).
- Now Component Tests check business logic: things like validation (all negative cases), all the equivalence classes, special symbols, etc.
- Unit tests.. Don't touch them - let them be written as before :)

*Pros:*
- Component Tests are way faster than System Tests. You'll decrease the number of System Tests, ergo your lead time will be smaller.
- Devs can even run Component Tests on their local machines. Very fast feedback.
- You won't need to bother too much with stability - because the number of System Tests will be kept low, spontaneous failures will be rare.

## How to implement Component Tests

Different eco systems will require different setups, but here are several examples:
- Spring MVC apps may leverage MockMvc framework to emulate requests from users.
- Apps that use some HTTP based protocol (REST/SOAP via HTTP) may use libs like [REST-assured](https://code.google.com/p/rest-assured/), [Restito](https://github.com/mkotsur/restito).
- Those who use MOMs like IBM MQ, Tibco EMS, Solace may want to use ActiveMQ as an alternative for Component Tests. Well, Devs who work with MOM usually write Component Tests anyway, so they know how to do it. Although they usually don't implement them in a readable way :)

It's important to realize that you don't have to bring up the whole App in order to run Component Tests - you may simply initialize what you need for a particular case.

For such tests it's usually preferred to use in-memory databases. There are tools that help implementing this: ORMs and DBs that can mimic other DBs (HSQLDB, TimesTen).

## Should AQA be involved?

My position on this - all the tests, no matter what level, have to be maintained by Devs. Manual QA may help coming up with test cases, they can also review existing tests (you have to write them in a readable form to let this happen) or even write tests themselves if you provide an easy-to-use business-oriented framework. AQA are more expensive and they still won't be able to implement business layers of the tests without Devs.

You may still keep AQA for Selenium stuff, but their results have to be carefully reviewed by Devs.

# Wrapping Up

As it was mentioned - I haven't tried component tests in Web Apps myself, but I'll update the article when this happen. I hope to see first results in a month.

If you're aware of next big steps of test evolution after Homo sapiens, please give me a shout.
