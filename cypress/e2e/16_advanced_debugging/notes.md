## Advanced debugging
End to end tests have a reputation for being flaky. But most of the times, the things we call "flakiness" are hidden timing issues in the app under test. Flakiness is annoying. If our tools could tell us the exact reason why a certain test is flaky, we would probably not talk about it so much. But most of the time, we are stuck with reporting tools, screenshots, videos, trace viewers and some other artifacts. These are all great tools and [I’m arguing that improvements of these tools](/next-big-trend-in-testing-debugging) is going to be very important in following years.

I like to say that flakiness is just lack of insight. Let’s take a simple example.

```js [spec.cy.ts]
cy.get('button').click()
cy.wait(5000)
cy.get('[data-test=item]').should('be.visible')
```

`cy.wait()` is the most basic way of fixing flakiness. We simply ask our test to stop executing, so that app can finish doing its thing. The real question is - what is the app doing? It can be waiting for responses from API, doing a calculation in a background, routing to another subpage, accessing browsers API or counting down a timeout.

The real reason why we are waiting is that we don’t know what is happening in our app. I guess if we had a stable app and stable server, our tests would **really** be more stable.

A more common problem problem with test flakiness are connected to timing issues. For example if the application under test performs two operations and does not handle the completion order. Let’s take a look at this example:

```jsx [App.tsx] {5-6}
const settingNumber = () => {
  const [number, setNumber] = useState(0);

  const createRace = () => {
    setNumber(1);
    setNumber(2);
  };

  return (
    <div>
      <button onClick={createRace}>Click me!</button>
      <p>Number: {number}</p>
    </div>
  );
};
```
As you can see, we call `setNumber()` twice. In a real world example you could imagine multiple API calls that update a part of your application. A double-click, server response lag, slower rendering performance or any other factor may have influence on the result. Since these functions are asynchronous, the number rendered in our `<p>` element may be 1 or 2. Or even better, it can jump from 1 to 2 and vice versa. 

Now imagine we are not dealing with number 1 or 2, but with rendering of a whole component or a page redirect.

Cypress deals with these types of situations pretty well. They utilize retryability, account for elements actionability and perform various checks to prevent flakiness. 

The example given is of course an oversimplification. Timing issues, re-rendering and other gotchas of modern web are going to stay with us for some time.

The real question is - how do we account for them? The answer both is simple and complicated: By getting more insight. If we want to battle flakiness, we need to get the missing insight and improve our test suite help, but also the application health.

[There’s a great blogpost](https://codingitwrong.com/2020/10/09/identifying-code-smells-in-cypress.html) from Josh Justice, that I keep coming back to. It talks about identifying problems within application by using e2e tests in Cypress. I feel like this is an ultimate goal of e2e testing.

We need to take the application under test into the equation. One way of doing that is using [Replay.io](https://replay.io). There are similarities between Cypress’ timeline and Playwright’s trace viewer. But Replay is a lot more. It’s basically 3 tools in one:
- browser
- recorder
- debugger

As a browser it can be used in both Cypress and Playwright. As a recorder, it will trace all interactions, DOM states, and network communication. But in addition to that it will capture the app’s source code and data flowing through it. As a debugger, it will allow you to travel back in time to any point of test execution and examine internal state of application. In the `createRace()` function from previous example we would be able to tell which `setNumber()` returned first or even see if the `<p>` element rendered multiple times.

The potential insight from this type of recording is much more robust and has the potential to provide with insight into timing issues or any kind of flake source.

## Creating a recording
To create a recording, download Replay.io browser, open it and hit the record button. After you do so, you can start interacting with your application as you would with a normal browser. For example, you can replicate a bug that you are experiencing in your app.

After you are done with interacting with your application, you will have your recording available. You can rewind or fast forward recording and look at different keystrokes, mouse clicks or other interactions. If you are in a process of replicating a bug, you can add comments to the recording and share it with developers in your team. This is where the real magic begins.

## Time travelling with DevTools
The fact that this recording can be done by anyone can lift a lot of weight off of developer’s shoulders. But creating the recording is just the beginning of the journey. Replay.io provides you with a set of devtools, that look very similar to Chrome or Firefox devtools.

But these are now attached to your recording. You can retroactively inspect elements, review API calls in the network panel, look at console logs and so much more.

## Debugging your tests
The recorded information is very useful, but it gets even better. As I mentioned in the beginning, Replay.io is actually a browser. Instead of replicating a bug manually, you can use your test run to create these recordings for you. This can be integrated to both Cypress and Playwright.

The setup is pretty simple. Replay has a Cypress plugin that works exactly like any other plugin. You install it as a package and include it in your `support/e2e.ts` file and in yout `cypress.config.ts`.

```bash
npm i @replayio/cypress
```

```ts [support/e2e.ts]
require('@replayio/cypress/support');
```

```ts [cypress.config.ts]
import { defineConfig } from "cypress";
import replay from "@replayio/cypress";

export default defineConfig({
  e2e: {
    setupNodeEvents(on, config) {
      replay(on, config);
      return config
    },
  },
});
```

After that, Replay.io needs to be set in your CI. I’m using GitHub Actions, but this can be set up in pretty much any CI provider.

```yml
name: Replay test
on: [push]
jobs:
  cypress-run:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2
      - name: Cypress run
        uses: cypress-io/github-action@v5
        with:
          browser: replay-chromium
          start: npm run dev
      - name: Upload replays
        if: always()
        uses: replayio/action-upload@v0.5.0
        with:
          api-key: ${{ secrets.RECORD_REPLAY_API_KEY }}
```

The API key can be obtained right from Replay.io browser and needs to be added as a secret to your GitHub project. If you are unfamiliar with how to set up GitHub Actions, I suggest you check out [my blog on this](https://filiphric.com/cypress-and-git-hub-actions-step-by-step-guide).