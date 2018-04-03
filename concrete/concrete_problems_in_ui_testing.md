# Concrete Problems in UI Testing

### Table of Contents
[Motivation](#motivation)\
[Pieces of the puzzle](#pieces-of-the-puzzle)\
[Quality of Test Results](#quality-of-test-results)\
[Scalability of UI Testing](#scalability)\
[Alternate Proposals](#alternate-proposals)

### Motivation
In the following article, I want to try and motivate the argument that large scale automation of UI tests won't let the SE product do continuous delivery, and also lay some groundwork for some future discussions about alternate strategies.

In order to motivate my argument, I want to first give a high level overview of the machinery that allows us to do automated UI testing, then talk about two potential problem classes that we tend to run into when trying to deploy automated UI tests on a large scale, and then end off with some ideas about how we might solve testing SE.

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

Now that we have this test case for our create-document functionality, IF this test case passes THEN our create-document functionality is definitely working, and the contrapositive also holds, right? Well actually that's necessarily true, and you may already have reasons in mind about what may go wrong, we will explore those ideas further later on.

### Quality of Test Results
Before we write any UI tests, a question that we might want to answer is why do we want to write UI tests, and from what I've gathered, our automated UI testing project was intended reduce the amount of resources we needed in manual testing by having automated test developers automate manual test cases so that manual wouldn't need to run them anymore. Our experience with this project has actually shown that this was not necessarily a very good idea, but since its problems are not specific to UI testing I'm not going to talk about them. The other argument that I've heard others in our team say is that we need UI tests because you can't really be sure that the application works unless it works from the user's perspective, which I would agree with in the sense that under practical circumstances that statement is probably true.

But then again there is a rebuttal, and that's where I talk about the reliability of UI testing, and that is reliability in two different respects in that there is reliability in the sense of to what degree can we trust our test results, as well as reliability in the sense of how often will a test suite blow up because the act of using WebDriver to control a browser through a script is essentially a non deterministic action from the perspective of the us the programmer.

To get back to what might be wrong with UI testing in our first section, lets try to answer the question of that IF our create-document test case passes THEN our create-document functionality is working and its contrapositive are true statements, because if its really easy to get into a situation where this doesn't hold, then perhaps we cannot trust the results of our UI tests.

Looking at the contrapositive case, IF the create-document test case fails Then the create-document functionality is not working. The most obvious way to try and make this untrue would be if the element selectors are no longer valid, then the test case will fail when you try to get the element, and the rebuttal to that would be a false negative is not a big deal, and you can just update the selector and the test will work as intended again. To that I would say that false negatives might be worse than you think, see boy who cried wolf, and a neural net that falsely classifies a patient's tumor as malignant, and while it is true that if your selector is not working you can just update it, we will see later that this is one thing that contributes to the scalability problems. Another way to get the contrapositive to not hold is if we decided that the document name should be displayed as ```${id} - ${name}``` instead of ```${id} ${name}``` and you might think that its similar to the problem we had with the outdated selectors, and you might know that these types of problems occur when you try to build an application without an API guarantee, but more on that later.

Now to look at how we might get a false positive in UI testing, this time around we are going to use an example of checking if the lockdown functionality for engagements is working. Lets say that the requirements for lockdown is that you can no longer edit the names, or create issues for documents. It might seem somewhat reasonable that we check if this requirement is fulfilled by seeing if the edit name and add issue buttons are there so after entering lockdown mode our checks are

```
assert.isFalse(editNameButton().isPresent())
assert.isFalse(createIssueButton().isPresent())
```

at first glance this seems reasonable, but if the selectors for editNameButton and createIssueButton change and then some time down the road the lockdown functionality stops working, this test case will still pass.

The obvious fix for this test case would be first assert that the two buttons were there, then enter lockdown mode, then assert that the two button aren't there anymore. And actually it won't actually doesn't work because I tried doing that and there is some convoluted reason why certain implementations for hiding the buttons won't work with the solution whether the solution works or not is not even that important. The important part is actually that its not obvious that the test case has this weakness in the first place because while we know the problem in hindsight, we've actually at one point our entire team came together and talked about this problem, and no one noticed it, and personally it wasn't until the third time that I worked on this code in this area that I noticed the problem. So in this case, this test case would have been giving us false positives if the lockdown functionality had been broken in any time in the last few months.

### Scalability of UI Testing
I think that there are mainly two problems we run into when we try to scale up UI testing to encompass an entire regression suite. The more one being the cost of owning UI tests, and the second being the time it takes to run the tests.

There are a few different ways to think about why the cost of owning UI tests are quite high, the more obvious way is to observe that UI tests only work on a specific implementation of the UI, so if the UI is altered there it is often necessary to update the test to reflect the changes, even if the functionality remains the same. However, a more helpful way to look at it that our UI tests are actually an application that is built on top of the API that is the UI of the SE platform, but what's different is that we don't have an API guarantee because the UI is always changing.

In general, sane developers would refuse to build an application on an API that might change, because when the their application gets sufficiently complicated, a seemingly small change in the API contract could completely brick their application. To give a concrete example of this, I would argue that our current UI tests application is actually constantly in various degrees of bricked, as at any given time about as quarter to half of our tests are not working, and this problem is exacerbated when new UI tests are continuously being added to our test suites. Due to the absurdly high cost of owning UI tests (the time it takes to maintain them and get broken test cases working again), in my own experience we are actually spending less than half of our work time on sprint work but instead trying to pay the cost of just having the tests.

The other problem with trying to scale UI testing is in a sense the time it takes to run the tests, but not exactly because if we could setup some UI testing system that could tell you if a build is good to release in 3 hours (the current time it takes to run our automated regression suite) I think it would actually be a pretty good deal. The problem actually come when you try to run a non deterministic system for 3 hours a pray that nothing will go wrong, which never actually happens because in practice about 10% of tests will fail just due to random non deterministic effects which will then require you to spend the next few hours trying to troubleshoot the failed tests and at this point its already incompatible with continuous delivery.

### Alternate Proposals
- hopefully my arguments about why the current way we are doing the thing is not compatible with thing CD thing
- use UI testing for smoke testing, use a test suite that is at most 100 (arbitrary number) tests
- we want the smoke test to run in at most 15 minutes, with some architecture improvements I think its possible.
- manual testing is for sanity testing the new features, and exploratory testing
- test development efforts can be redirected to unit test development, the only integrated testing that should be done is if client code can talk to the server properly, done from a run time level.
