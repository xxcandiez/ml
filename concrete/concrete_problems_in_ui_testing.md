# Concrete Problems in UI Testing

### Table of Contents
[Motivation](#motivation)\
[Pieces of the puzzle](#pieces-of-the-puzzle)\
[Toy Example](#toy-example)\
[Quality of Test Results](#quality-of-test-results)\
[Scalability of UI Testing](#scalability)\
[Alternate Proposals](#alternate-proposals)

### Motivation
In the following article, I want to try and motivate the argument that large scale automation of UI tests won't let the SE product do continuous delivery, and also lay some groundwork for some future discussions about alternate strategies.

In order to motivate my argument, I want to first give a high level overview of the machinery that allows us to do automated UI testing, then talk about two potential problem classes that we tend to run into when trying to deploy automated UI tests on a large scale, and then end off with some ideas about how we might solve testing SE.

### Pieces of The Puzzle
All of our UI tests are built on top of WebDriver, which is a wire protocol that can be used to control a web browser remotely through a script (or just a cli). ChromeDriver implements the WebDriver protocol for the Chrome browser, Selenium provides JavaScript language bindings to a WebDriver API, and Protractor uses Selenium's JavaScript bindings and adds some Angular specific features. We use Protractor to use the WebDriver protocol to control the browser by running scripts in a JavaScript runtime.

Now that we can control a browser from a JavaScript runtime, we might think that we should consider writing automated end-to-end tests on our application. Using the WebDriver protocol, our tests follow the general pattern of, do some setup by navigating to pages, or clicking on buttons, or filling in forms, and then inspect the DOM and make assertions based on the state of it.

### Toy Example

To give a better idea about how we can use these tools lets say that we want to check if the create-document functionality in SE is working. Assuming that you are in the documents page of SE, we can choose which DOM elements we want to interact with by using CSS selectors, so we can first declare some variables that will be our DOM elements.

To help us talk about automated UI testing more clearly a toy example of an automated UI test that tests if one acceptance criteria of engagement lockdown is working, we can pretend that the code works as intended.

To work this example we're first going to define 3 classes, DocumentRow, Navbar, and EngagementProperties, we say that Navbar contains EngagementProperties. You can skip the class declarations don't feel like going through the code.

##### Class Declarations

```
class Navbar {
  get root () {
    return $('#navbar')
  }

  get EngagementProperties () {
    return new EngagementProperties(this.root.$('#engagementPanelContainer'))
  }
}

class EngagementProperties () {
  constructor (root) {
    this.root = root
  }

  get engagementPanelToggle () {
    return this.root.$('#engagmentPanelToggle')
  }

  get engagementPanel () {
    return this.root.$('#engagementPanel')
  }

  get lockdownButton () {
    return this.root.$('#lockdownButton')
  }

  async openEngagementPanel () {
    let opened = await this.engagementPanel.isOpen();
    return !opened && await engagementPanelToggle.click();
  }

  async enterLockdownMode () {
    await this.openEngagementPanel()
    return await this.lockdownButton.click();
  }
}

class DocumentManager {
  get root () {
    return $('#documentManager')
  }

  getDocumentRowByName(name) {
    return this.root.element(by.cssContainingText('#document', name))
  }
}

class DocumentRow {
  constructor (root) {
    this.root = root
  }

  get editButton() {
    return this.root.$('#editButton')
  }

  get issuesButton() {
    return this.root.$('#issuesButton')
  }
}
```

##### Test

```
describe('lockdown', () => {
  it('should disable editing and adding issues to documents after lockdown', async () => {
      // declare stuff we need
      let defaultDocument = new DocumentManager.getDocumentRowByName('default document')
      let engagementPanel = new Navbar().engagementPanel

      // setup
      await engagementPanel.openEngagementPanel()
      await engagementPanel.lockdownButton.click()

      // inspect the DOM
      assert.isFalse(await defaultDocument.editButton.isPresent())
      assert.isFalse(await defaultDocument.issuesButton.isPresent())
    })
})
```

This test case would probably be used to check the acceptance criteria "users should not be able to edit or add issues to documents in lockdown mode." What this test does is it keeps track of the state of the edit button and issues button of a DocumentRow in the document manager, it checks that in lockdown mode there is no edit or issues button for the user to use.

At first glance, our test seems like a pretty reasonable way to see if that acceptance criteria of the lockdown functionality is working, but it turns out that its really easy to get into a situation where this test will actually give us both false positives and false negatives, which if true would mean that we would either have to spend test development's time to investigate a non existent issue, or would result in shipping a potential bug or untested feature.

##### Aside
While testing we want to took look for bugs, which presumably correspond to failed test cases, so our positive classification is bug and negative is no bug. In the context of test run results I want to define a false positive as a test case that has failed when the criteria that it intended to check is still fulfilled, and false negative as a test case that has passed when the criteria that it intended to check is not fulfilled, _or if the test is in a tautology state where it will always pass._

### Quality of Test Results

For now I don't want to talk about false positives too much because other than a test failure due to some random non determinism, the only other why that I can think of that a test would give a negative is that there is some change in the UI, which the answer to that is to update the CSS selectors, or the navigation in the test scripts.

A feature of UI tests is that the test scripts need to be periodically updated, when there are UI changes, a certain number of our test cases be pushed into contradiction states, those test cases will fail and we will learn about then though our daily test run results. But with UI changes some test cases are pushed into tautology states where from now on they will always pass, and there is not really any practical way to detect them.

To give a more concrete example we can use our toy example to see how a test case can enter such a state. Here is our toy example.

```
let defaultDocument = new DocumentManager.getDocumentRowByName('default document')
let engagementPanel = new Navbar().engagementPanel

await engagementPanel.openEngagementPanel()
await engagementPanel.lockdownButton.click()

assert.isFalse(await defaultDocument.editButton.isPresent())
assert.isFalse(await defaultDocument.issuesButton.isPresent())
```

We can say that the test is working as intended because if the edit button or issues button were still present after lockdown mode was entered, our test case will fail. However in the next iteration of SE there is a feature where after you lockdown the engagement, you are redirected to a page that contains a summary of all work that was done on the engagement. The test case is pushed into a tautology state after the update because there is always going to be no edit button or issues button on the summary page. But since we are programmers, and we are very smart, we knew that this might happen so we actually wrote the original test case like this.

```
let defaultDocument = new DocumentManager.getDocumentRowByName('default document')
let engagementPanel = new Navbar().engagementPanel
let url = await driver.getUrl()

await engagementPanel.openEngagementPanel()
await engagementPanel.lockdownButton.click()

// if the url has changed, go back to the original page
let newUrl = await driver.getUrl()
if (url !== newUrl) {
  driver.goto(url)
}

// check that we are on the right page just to be safe
assert.isEqual(url, await driver.getUrl())
assert.isFalse(await defaultDocument.editButton.isPresent())
assert.isFalse(await defaultDocument.issuesButton.isPresent())
```

Our updated test case can actually still enter a tautology state when there is a new update, because the update changed the selectors for the edit and issues buttons, and even if the new button are there are can still be used in lockdown mode, our test says that the old buttons aren't present so everything is fine.

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
