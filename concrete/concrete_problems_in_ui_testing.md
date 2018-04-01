# Concrete Problems in UI Testing

### Table of Contents
[Motivation]()<br>
[Pieces of the puzzle]()<br>
[What does it even do/isDisplayed]()<br>
[Reliability/Continuous Delivery]()<br>
[Scalability/Effort/Value prop/interface vs implementation/absurdity of application on application]()<br>
[cost]()<br>
[Alternate Proposals]()<br>

### Motivation
- In this following article I will try to motivate some of the problems with UI testing
- cost of ownership and adding new test cases every sprint
- incompatibility with continuous delivery (reliability and run time)

In the following article I want give a high level overview of how UI testing works, and also motivate some problems with UI testing, some reasons why our UI tests won't necessarily be compatible with continuous delivery, and some possible alternatives we can consider to UI testing for CD.

### Pieces of The Puzzle (How does it work?)
- WebDriver is a wire protocol that's a w3c proposal
- the protocol is designed to let a user remotely control a web browser
- unsurprisingly some people thought that perhaps the WebDriver protocol could be used to automate UI testing and rightfully so because the WebDriver protocol can be a very useful tool for some aspects of test automation, but as we will see, it can also be horribly misused and cause a lot of problems just like object orientation but that's for another time.

- We write our tests using protractor which is a JS library that wraps a bunch of other libraries which eventually wraps an implementation of WebDriver chrome driver, this allows us to execute our test scripts from a node environment.

- using the wire protocol we can perform actions such as find elements, click elements, fill forms.
- so to test, we do some actions on the elements then we inspect the dom

### Reliability of UI tests
- first question is why one might want to write UI tests
- idea is, to figure out if our application is actually working properly, we should run end to end tests, so that our tests actually run from the perspective of one of our users, which is actually not too bad of an idea if you get a human to run the tests
- why might a UI test fail when the application still "works as intended"
- why might a UI test pass when the application does not work as intended
- one of the things we might decide to trade off when we do UI tests is perhaps cleanliness for the hope that we don't run into a situation where all of our tests pass but we still have a bug in the code, as we can see UI testing doesn't even solve that
- race conditions causes innate flakiness, point out that google engineers also report significant flakiness in their tests, its not a trivial task to say, just write your tests so that they are reliable, fan fan pointed out that the sleep times in WebDriver is not tied to how fast the browser running, its just a static sleep time.

### Scalability
- why would you want to couple your application to a test suite
- absurdly long run times
- does not react well to change, UI testing essentially involves writing an application on top of your existing application, but unlike writing an application on top of an API, you actually have no guarantees about how the application is going to behave
- essentially like writing an application with no API guarantee
- a concrete example of this is when the done button was replaced by a toggling edit button
- an example of a small change that would actually completely brick our project, is if you changed the behaviour of entering a newly created document to not enter into edit mode by default, since there are so many places in our scripts that implicitly rely on the fact that it does
- something changed in checklists from a DOM perspective and its actually going to take multiple weeks to fix.
- the reason why this thing happens in UI testing is because of interface vs implementation testing.
- the way that we check if the add document functionality works is by pressing some buttons then checking if a ceretain list in the DOM now contains the text of the new document name. this is similar to testing a add function for instance by checking that the function has a return statement, takes two parameters, and uses the '+' symbol once in the source code, it is just completely absurd
- how is this going to work in conjunction of the reliability problem in CD, if we wait for the tests to run for 4 hours, then let the QA team investigate the failures for the 40 failed tests over the next 2-3 days, that's not exactly CD.

- cost of maintenance, these properties of UI tests make them so expensive to do
- its one of the reasons why I always seem apprehensive to add more UI tests into our test suites, because the cost of maintaining each one is so high, and the cost of maintenance for each test cases becomes higher the more test cases you have, I honestly believe that I am doing more harm than good by adding new test cases into our test suites.
- cost of babysitting your test cases, (new dev put it really nicely, essentially a manual task)
- from a development perspective slow feedback times, (1-5 mins), there is so much wasted time in UI test development

### Alternatives
- hopefully my arguments about why the current way we are doing the thing is not compatible with thing CD thing
$\lambda$
