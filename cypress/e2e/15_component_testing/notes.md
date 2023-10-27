# Component testing

## What is a component?
I choose to look at them as lego blocks. In web development, these lego blocks are what a web application is built of. Lego bricks come in different shapes and sizes and can serve different purposes. Some lego block are used just once, some are reused all the time. Same goes for components!

Each component has certain visual and functional properties. Like a lego block, a component can be a common one, that get’s used all the time and changed just slightly. Or it can be a unique one, that serves a specific function.

Let‘s take a look at a simple Vue component:

```vue
<template>
  <button class=".green-button"></button>
</template>

<script setup lang="ts">
defineProps({
  buttontext: {
    type: String
  }
});
</script>
```
In my application, I can import the component anywhere I need it, and use it like this:
```vue
<MyButton buttontext="Hello world" />
```
The button will render differently based on what property is passed into it.

## Why test a component?
If you need to make sure that this button looks good with an emoji, special character or a normal text, what do you do? Going with e2e approach - clicking through our app to get to it is not going to be effective.
​
Enter component testing.
​
Testing a component in isolation gives you a much wider set of options. Imagine you want to refactor your component. While you are doing so, you can get an immediate feedback on anything that might be broken. In other words, it’s easier to just render a component with an emoji, than go through your app looking for it.

This is why component testing is growing in popularity and tools like Jest, Storybook and now Cypress are widely used.

## How to set up
With Cypress v10 set up of component testing is very simple. Actually, Cypress authors have made a great job making it as simple as possible.

Once you open Cypress GUI using `npx cypress open --component` command, you will be welcomed by by setup screen where you can set up your framework and your bundler. 

As a tester, I needed to get myself familiar with these terms, especially with "bundler". What the bundler does, is that it converts the code you write into something a browser can read. As mentioned in last newsletter issue, components can help us with splitting our application into small "lego blocks". These are often `.vue` files, or `.jsx` files or something similar. But these will not work in browser by themselves. Bundler makes them into a bunch of `.html`, `.js` and `.css` files that browser can read.

When running a component test, Cypress will use the bundler to convert your component file into something a browser can read. Using the same bundler as your application does.

## Cypress component testing project
Based on the inputs, installation wizard will set up our project. Looking at the cypress.config.ts file you can see that the configuration is actually pretty simple:

```ts
export default defineConfig({
  component: {
    devServer: {
      framework: "vue",
      bundler: "vite",
    },
  },
});
```

By default, Cypress will take options set in `vite.config.ts` file in our project, so anything we have set up for our app, will instantly become available to Cypress tests as well.

## cy.mount()
Besides resolving our configuration, Cypress will create a couple of helper files, one of the most important being component.ts located in `cypress/support` folder.

```ts
import { mount } from 'cypress/vue'

  // Augment the Cypress namespace to include type definitions for
  // your custom command.
  // Alternatively, can be defined in cypress/support/component.d.ts
  // with a <reference path="./component" /> at the top of your spec.
  declare global {
    namespace Cypress {
      interface Chainable {
        mount: typeof mount
      }
    }
  }
  
  Cypress.Commands.add('mount', mount)
  
  // Example use:
  // cy.mount(MyComponent)
```

This file will contain the mounting function depending on the framework you use. In this email course, we will be working with Vue, but Cypress has already support for React, Angular, Svelte and even frameworks like Next.js and Nuxt. More are coming with every release and most of them will come out of BETA any day now.

Your mount function is wrapped in a custom command. This enables you to customize it in many ways, and we will be talking about different ways of customizing the command in future issues.

## How Cypress is built
Cypress’ architecture is sort of unique when compared to other testing tools. With Playwright or Selenium, the goal is to automate a browser in order to perform some actions. With Cypress, you are essentially building an app to test your app.

When you are developing an application, your files get bundled and opened inside a browser. Imagine your standard npm run dev mode.

With Cypress, pretty much the same principle is applied. Your test files get bundled and openened in a browser. With component testing, instead of opening the app for your end to end test, you’ll mount your component. Pretty cool if you ask me.

