---
title: Everything you need to know about React 18
cover: assets/react-18.png
date: "2022-02-23"
authorName: Ashik
authorUrl: https://ashrhmn.com
authorPosition: Blockchain Intern
published: true
categories: ["React","JavaScript","Frontend"]
desc: React is a free and open-source front-end JavaScript library for building user interfaces based on UI components
toc:
  [
    { name: "Root API", id: "root-api" },
    { name: "React 17", id: "react-17" },
    { name: "React 18", id: "react-18" },
  ]
---

React 18 alpha version was just announced. The theme of React 18 is to make the UI more performant by removing janky user experiences by introducing out of the box features and improvements powered by "concurrent rendering". React 18 introduces minimal breaking changes.

Let's take a look at the major updates of React 18:

## Root API

React 18 introduces Root API `ReactDOM.createRoot`. Before React 18, we used ReactDOM.render to render a component to the page. Going forward with React 18, we will use ReactDOM.createRoot to create a root, and then pass the root to the render function. The good news is that your current code with `ReactDOM.render` will still work, however, it is strongly recommended to start transitioning to createRoot as render will be marked deprecated starting React 18. The current `ReactDOM.render` is only provided to ease the transition to React 18.

### React 17

```js
import ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

ReactDOM.render(<App />, container);
```

### React 18

```js
import ReactDOM from 'react-dom';
import App from 'App';

const container = document.getElementById('app');

// create a root
const root = ReactDOM.createRoot(container);

//render app to root
root.render(<App />);
```

## Automatic batching ( Out of the box, opt-out available):

Batching is the process of grouping multiple state updates into one to prevent multiple re-renders. Previously, React batched state updates that happened in a single event callback managed by React event system, however state updates that happened outside the event were not batched.

However, with automatic batching, all updates, even within promises, setTimeouts, will be batched. Check this example -

```js
function handleClick() {
    console.log("=== click ===");
    setCount((c) => c + 1); // Does not re-render yet
    setFlag((f) => !f); // Does not re-render yet
    // React will only re-render once at the end (that's batching!)

    const timeoutCallback = () => {
      // Previously, batching didn't work inside timeouts, fetch, promises.
      // These two setStates causesd re-render in React 17.
      // With React 18, these are now batched.
      setCount((c) => c + 1);
      setFlag((f) => !f);
    };

    setTimeout(timeoutCallback, 1000);
  }
  ```

  **Opt-out:** You can opt-out of automatic batching by using `flushSync`

  ## startTransition (opt-in)

`startTransition` can be used to mark UI updates that do not need urgent resources for updating. For eg: when typing in a typeahead field, there are two things happening - a blinking cursor that shows visual feedback of your content being typed, and a search functionality in the background that searches for the data that is typed.

Showing a visual feedback to the user is important and therefore urgent. Searching is not so urgent, and hence can be marked as non-urgent. This is where `startTransition` comes into play.

By marking non-urgent UI updates as "transitions" , React will know which updates to prioritize making it easier to optimize rendering and get rid of stale rendering. Updates marked in non-urgent `startTransition` can be interrupted by urgent updates such as clicks or key presses.

```js
import { startTransition } from 'react';


// Urgent: Show what was typed
setInputValue(input);

// Mark any state updates inside as transitions
startTransition(() => {
  // Transition: Show the results
  setSearchQuery(input);
});
```

### How is it different from debouncing or setTimeout?

1. startTransition executes immediately unlike setTimeout. setTimeout has a guaranteed delay, whereas startTransition's delay depends on the speed of the device, and other urgent renders.
2. startTransition updates can be interrupted unlike setTimeout and won't freeze the page.
3. React can track the pending state for you when marked with startTransition.