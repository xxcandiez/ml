# Concrete Problems in UI Testing

### Table of Contents
[Motivation](#motivation)\
[Pieces of the puzzle](#pieces-of-the-puzzle)\
[Toy Example](#toy-example)\
[Quality of Test Results](#quality-of-test-results)\
[Scalability of UI Testing](#scalability)\
[Conclusions and Ideas](#conslusions-and-ideas)

### Motivation
In the following article, I want to motivate some arguments about why large scale automated UI tests won't let the SE product do continuous delivery, and also lay some groundwork for some future discussions about alternate testing strategies.

In order to motivate my argument, I want to first give a high level overview of the machinery that allows us to do automated UI testing, then talk about two potential classes of problems that we tend to run into when trying to deploy automated UI tests on a large scale, and then end off with some ideas about how we might approach testing SE.

### Pieces of The Puzzle
Our UI tests require WebDriver, which is a wire protocol that can be used to control a web browser remotely through a script (or just a cli). ChromeDriver implements the WebDriver protocol for the Chrome browser, Selenium provides JavaScript language bindings to a WebDriver API, and Protractor uses Selenium's JavaScript bindings and adds some Angular specific features. We use Protractor to use the WebDriver protocol to control the browser by running scripts in a JavaScript runtime. We write tests by following the general pattern of doing some setup by navigating to pages, clicking on buttons, or filling in forms, and then inspecting the DOM and making assertions based on what we expect about the state of it.

### Toy Example
To help us talk about automated UI testing more clearly, we are going to use a toy example of an automated UI test that checks if engagement lockdown is working. To work this example we're first going to define 3 classes, DocumentRow, Navbar, and EngagementProperties, we say that Navbar contains EngagementProperties and DocumentRow is just the element that is the placeholder for a document in the DocumentManager. You can skip the class declarations if you don't feel like going through the code.

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

This test case can be used to check the acceptance criteria "users should not be able to edit or add issues to documents in lockdown mode." What this test does is it sets up the scenario by putting the document in lockdown mode, then it checks if lockdown mode is working by asserting that both the edit button and issues button for the document row is no longer there.

At first glance, our test seems like a pretty reasonable way to check if the acceptance criteria of the lockdown functionality is satisfied, but it turns out that its really easy to get into a situation where this test will actually give us false positives or false negatives, which if true would mean that we would spend development time to investigate a non existent issue in the false positive case, or ship a potential bug or untested feature in the false negative case.

##### Aside
While testing we want to look for bugs, which presumably correspond to failed test cases, so our positive classification is bug and negative is no bug. In the context of test run results I want to define a false positive as a test case that has failed when the criteria that it intended to check is still fulfilled, and false negative as a test case that has passed when the criteria that it intended to check is not fulfilled, _or if the test is in a tautology state where it will always pass._

### Quality of Test Results

For now I don't want to talk about false positives too much because other than a test failure due to some random non determinism, the only other way that I can think of why a test would give a false positive is that there is some change in the UI, which the solution to that is to "just update the test scripts."

A feature of UI tests is that the test scripts need to be periodically updated, that's because when there are any UI changes, a certain number of test cases are pushed into contradiction states, those test cases will always fail and we will learn about them through our daily test run results, those are our false positive results. But with UI changes some test cases are actually pushed into tautology states, where from now on they will always pass no matter what. Those tests are not reported in our test results, and there is not really any practical way to detect them, they will remain hidden until a test developer randomly stumbles upon that test case and finds the mistake.

To give a more concrete example we can use our toy example to see how a test case can enter such a state. Here is our toy example.

```
let defaultDocument = new DocumentManager.getDocumentRowByName('default document')
let engagementPanel = new Navbar().engagementPanel

await engagementPanel.openEngagementPanel()
await engagementPanel.lockdownButton.click()

assert.isFalse(await defaultDocument.editButton.isPresent())
assert.isFalse(await defaultDocument.issuesButton.isPresent())
```

We can say that the test case is working as intended right now because if the edit button or issues button were still present after lockdown mode was entered, our test case will fail. However in the next iteration of SE there is a feature where after you lockdown the engagement, you are redirected to a page that contains a summary of all work that was done on the engagement. The test case is pushed into a tautology state after the update because there is always going to be no edit button or issues button on the summary page. But since we are programmers, and we are very smart, we knew that this might happen so we actually wrote the original test case like this.

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

However, our updated test case can actually still enter a tautology state when there is a new update, because the update changed the selectors for the edit and issues buttons, and even if the new buttons are there, and can still be used in lockdown mode, our test says that the old buttons aren't present so everything is fine.

### Scalability of UI Testing
When it comes to problems about scaling automated UI tests, I think that the largest problem is that the cost of maintenance quickly becomes unsustainable, this is mostly due to the need to rewrite test cases on UI updates. To get back to the point I made earlier that to fix a false positive test case you just need to update the scripts, the problem is that updating the scripts is actually not straight forward, and it becomes more difficult to update the scripts the larger your test repository is.

If you look back to the class declarations for our toy example, you may notice that we are essentially building controllers from the user's perspective on top of the application. The problem comes when the UI is updated and you need to update the scripts, but its not a trivial task because there are assumptions about the UI that our controllers make that are so deeply built into our code, so even seemingly innocuous changes to the UI can require an absurdly long time to account for in test development.

The reason why this problem gets worse with a larger test repository is because our tests and associated controllers are somewhat synonymous to an application that is built on top of an API that randomly changes. Depending on how large your application is or how much of your application relies on the assumption that the API behaves in a certain way, its possible that the change can completely brick your application, and it would be harder to recover from an API change when your application is larger or more complex.

### Conclusions and Ideas
If you subscribe to my arguments, it would seem that trying to scale up the automated UI testing actually doesn't work with continuous delivery because there is still a bottleneck in testing. Due to all of the UI changes that happen during a sprint, anywhere from 25-50% of our test scripts are not functional. In the best case scenario where the tests are all functional still about 10% of them will fail from random non deterministic effects and will have to be manually verified. If we can solve the bottleneck problems, we will still ship obvious bugs because the test results are unreliable since we have no feasible way to detect false negative results.

I think that automated UI test can still be useful for smoke testing, because given its problems in scalability perhaps a small suite of about 100 (completely arbitrary number) test cases can be used to test some core functionality through the UI without requiring too many resources. For testing the rest of the application I think that its really important to have a test suite that is fast, and doesn't require a large amount of time put into maintenance just to have a reasonable portion of the tests to work. I think that a reasonable option would be to implement a series of unit and integration tests, I would recommend a talk by J.B. Rainsberger, [Integrated Tests Are a Scam](https://vimeo.com/80533536), for more details.
