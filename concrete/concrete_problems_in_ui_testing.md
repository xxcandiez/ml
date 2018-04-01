# Concrete Problems in UI Testing

### Table of Contents
[Motivation]()
[Pieces of the puzzle]()
[What does it even do/isDisplayed]()
[Reliability/Continuous Delivery]()
[Scalability/Effort/Value prop/interface vs implementation/absurdity of application on application]()
[Alternate Proposals]()

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
- so to test, we do some actions on the elements tben we inspect the dom

### haHAA
