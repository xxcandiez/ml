# Concrete Problems in UI Testing

### Table of Contents
[Motivation](#motivation)\
[Pieces of the puzzle](#pieces-of-the-puzzle)\
[Reliability and Continuous Delivery](#reliability-and-continuous-delivery)\
[Scalability of UI Testing](#scalability)\
[Alternate Proposals](#alternate-proposals)

### Motivation
In the following article I want give a high level overview of how UI testing works, and also motivate some problems with UI testing, some reasons why our UI tests won't necessarily be compatible with continuous delivery, and some possible alternatives we might want to consider.

### Pieces of The Puzzle
WebDriver is a wire protocol that can be used to control a web browser remotely, or through a script. ChromeDriver implements the WebDriver protocol for the Chrome browser, Selenium provides JavaScript language bindings to a WebDriver API, and Protractor uses Selenium's JavaScript bindings and adds some Angular specific features. Using Protractor we can use the WebDriver protocol to control the browser by running scripts in a JavaScript runtime.

Armed with the knowledge that we can control a browser from a JavaScript runtime, we might think that we should consider writing automated end-to-end tests on our application. This sounds like a really good idea, but just like object orientation it can also be horribly misused and cause unpleasant things to happen, we will explore this idea further.

Using the WebDriver protocol, our tests follow the general pattern of, do some setup by navigating on pages or clicking on buttons, or filling in forms, then inspect the DOM and make assertions based on the state of the DOM.

To give a better idea about how we can use these tools lets say that we want to check if the create-document functionality in SE is working. Assuming that you are in the documents page of SE, we can choose which DOM elements we want to interact with by using CSS selectors, so we can first declare some variables that will be our DOM elements.

```
const name = 'myName'
const id = 'myId'

// these are hypothetical selectors that we might use
//in practice we would probably declare these as getters in some class
const newDocumentButton = () => $('#newDocumentButton')
const checklistOption = () => $('.docuemntAdder.checklist')
const nameField = () => $('.documentName')
const idField = () => $('#documentId')
const okButton = () => $('[onClick="createDocument"]')
```

Now that we have our selectors, we need to perform the action of creating a new document.

```
// WebDriver actions are implemented as promises in Protractor
await newDocumentButton().click()
await checklistOption().click()
await nameField().clear().sendKeys(name)
await idField().clear().sendKeys(id)
await okButton().click()

await driver.sleep(1000)// wait for server to create the new document
```

Assuming that nothing went wrong, we now arrive in an application state where there is a new document in our engagement so we can do a DOM inspection to see if our create-document functionality worked, for a questionable definition of worked.

```
const documentNamesContainer = () => $('#allDocuments')// element that contains of all document names in text

assert.isTrue(documentNamesContainer().getText().contains(`${id} ${name}`))
```

Now that we have this test case for our create-document functionality, IF this test case passes THEN our create-document functionality is definitely working, and the contrapositive also holds, right? Well actually that's necessarily true, and you may already have reasons in mind about what may go wrong, we will explore those ideas further in the next section.

### Reliability and Continuous Delivery
Before we write any UI tests, a question that we might want to answer is why do we want to write UI tests, and from what I've gathered, our automated UI testing project was intended reduce the amount of resources we needed in manual testing by having automated test developers automate manual test cases so that manual wouldn't need to run them anymore. Our experience with this project has actually shown that this was not necessarily a very good idea, but since its problems are not specific to UI testing I'm not going to talk about them. The other argument that I've heard others in our team say is that we need UI tests because you can't really be sure that the application works unless it works from the user's perspective, which I would agree with in the sense that under practical circumstances that statement is probably true.

But then again there is a rebuttal, and that's where I talk about the reliability of UI testing, and that is reliability in two difference respects,

- incompatibility with continuous delivery (reliability and run time)
- unreliability and run time

- first question is why one might want to write UI tests
- idea is, to figure out if our application is actually working properly, we should run end to end tests, so that our tests actually run from the perspective of one of our users, which is actually not too bad of an idea if you get a human to run the tests
- why might a UI test fail when the application still "works as intended"
- why might a UI test pass when the application does not work as intended
- one of the things we might decide to trade off when we do UI tests is perhaps cleanliness for the hope that we don't run into a situation where all of our tests pass but we still have a bug in the code, as we can see UI testing doesn't even solve that
- race conditions causes innate flakiness, point out that google engineers also report significant flakiness in their tests, its not a trivial task to say, just write your tests so that they are reliable, fan fan pointed out that the sleep times in WebDriver is not tied to how fast the browser running, its just a static sleep time.

### Scalability of UI Testing
- cost of ownership and adding new test cases every sprint
- Scalability/Effort/Value prop/interface vs implementation/absurdity of application on application

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
- from a development perspective slow feedback times, (1-5 minutes), there is so much wasted time in UI test development

### Alternate Proposals
- hopefully my arguments about why the current way we are doing the thing is not compatible with thing CD thing
- use UI testing for smoke testing, use a test suite that is at most 100 (arbitrary number) tests
- we want the smoke test to run in at most 15 minutes, with some architecture improvements I think its possible.
- manual testing is for sanity testing the new features, and exploratory testing
- test development efforts can be redirected to unit test development, the only integrated testing that should be done is if client code can talk to the server properly, done from a run time level.
