# Concrete Problems in UI Testing

### Table of Contents
[Motivation]()<br>
[Pieces of the puzzle]()<br>
[What does it even do/isDisplayed]()<br>
[Reliability/Continuous Delivery]()<br>
[Scalability/Effort/Value prop/interface vs implementation/absurdity of application on application]()<br>
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
- race conditions

### Scalability
- why would you want to couple your application to a test suite
-
