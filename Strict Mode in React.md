Hello React developers, you've probably heard of strict mode. But what the hell is it, exactly? In short term, `React.StrictMode` is a useful component for highlighting potential problems in an application. Just like `<Fragment>`, `<StrictMode>` does not render any extra DOM elements.. With this article, we'll dive into the details of what strict mode is, how it works and why you should consider using it in your applications.

## What is Strict Mode in React?

Strict mode is a set of development tools that help you catch potential problems in your code before they become actual bugs. When you enable strict mode in your React application, you're essentially telling React to turn on a bunch of extra checks👀 and warnings that are designed to help you write better code. These checks and warnings can catch things like:

1. Components with side effects
2. Deprecated or unsafe lifecycle methods
3. Unsafe use of certain built in functions
4. Duplicate keys in lists

![react-strict-mode](https://res.cloudinary.com/practicaldev/image/fetch/s--0DFhwYBf--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/10tfwojwoyip5ld20yec.gif)

## Enabling Strict Mode

Enabling strict mode in your React application is actually very easy. You can do it by adding a single line of code to your main `index.js` file. Let's see:

```js
import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>,
  document.getElementById('root')
);
```

## Enabling strict mode for a part of the app

You can also enable Strict Mode for any part of your application:

```js
import { StrictMode } from 'react';

function App() {
  return (
    <>
      <Header />
      <StrictMode>
        <main>
          <Sidebar />
          <Content />
        </main>
      </StrictMode>
      <Footer />
    </>
  );
}
```

In this instance, Strict Mode checks will not run against the `Header` and `Footer` components. But, they will run on `Sidebar` and `Content`, as well as all of the components inside them, no matter how deep it is.

## Strict mode affects only the development environment

It's important to note that strict mode has no effect on the production build of your React application. This means that any checks or warnings that are enabled by strict mode will not be present in the final version of your application that users see. As we have seen, strict mode turns on extra checks and warnings, it can potentially slow down your development environment.

![Strict mode only in production](https://res.cloudinary.com/practicaldev/image/fetch/s--3O611kp_--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_66%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nu5kpm4ixxfesjdhv1cv.gif)

## Strict mode can help you catch subtle bugs

Sometimes bugs in React applications can be difficult to track down, especially if they're caused by subtle issues like race conditions or incorrect assumptions about component state. By enabling strict mode and taking advantage of the extra checks and warnings it provides, you may be able to catch these bugs before they cause serious problems.

## Strict mode can help you stay up-to-date with best practices

React is a rapidly evolving framework and best practices can change over time. By enabling strict mode and paying attention to the warnings and suggestions it provides, you can ensure that your React code is up-to-date and following current best practices. Such as using the `key` prop when rendering lists or avoiding side effects in `render()`.

## Highlighting potential problems early

Strict mode can catch issues in your code that may cause problems in coming future, before they become serious problems. Let's say, it can detect and warn about deprecated lifecycle methods as well as access to [findDOMNode()](https://react.dev/reference/react-dom/findDOMNode) which is no longer recommended.

![find-dom-node-depr](https://res.cloudinary.com/practicaldev/image/fetch/s--b_u3jpXu--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/awx9dnh9eexg4eb73wt2.png)

![class-component-snippet](https://res.cloudinary.com/practicaldev/image/fetch/s--M3abKvzy--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/i5hjs6oae3lj38mbhjwk.png)

## Preventing common mistakes

By enabling strict mode, you can avoid common mistakes that may not be immediately obvious, such as modifying the state directly instead of using `setState()` or using `undeclared` variables.

## Identifying Unsafe Lifecycles

React Strict Mode can help identify the use of unsafe lifecycle methods in your components. For instance, it can warn you about the use of the `componentWillUpdate` and `componentWillReceiveProps` methods which are both considered unsafe and will be removed in future versions of React.

Let's say you have a component that uses `componentWillReceiveProps()` to update its state based on incoming props:

```js
class BlackMamba extends React.Component {
  constructor(props) {
    super(props);
    this.state = { venom: props.venom };
  }

  componentWillReceiveProps(nextProps) {
    this.setState({ venom: nextProps.venom });
  }

  render() {
    return <div>{this.state.venom}</div>;
  }
}
```

With regular mode, this component would work as expected, but in `React.StrictMode`, you would see a warning in the console:

> Warning: BlackMamba uses `componentWillReceiveProps`, which is considered an unsafe lifecycle method. Instead, use static `getDerivedStateFromProps`.

## Warning About Legacy String ref API Usage

Strict Mode can also warn you about the use of the legacy string `ref` API, which is considered deprecated. But, you should use the `createRef` API, which is safer and easier to use.

![ref-api-usage](https://res.cloudinary.com/practicaldev/image/fetch/s--uwtb6lC4--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gv8lf8ubk7vv4yg8m4qo.png)

Let's say, you have a component that uses a string ref like this:

```js
import { Component } from "react";
import { createRoot } from "react-dom/client";

class BlackMamba extends Component {
  componentDidMount() {
    this.refs.venom.focus();
  }

  render() {
    return (
      <>
        <h1>Black Mamba</h1>
        <input ref="venom" />
      </>
    );
  }
}

const root = createRoot(document.getElementById("root"));

root.render(<BlackMamba />);
```

Legacy string `ref` API is considered deprecated, still you'll not see any errors in console.

![codesandbox-snippet](https://res.cloudinary.com/practicaldev/image/fetch/s--9wk8SBMH--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/trxw9auusgunit73rvzp.png)

When you enable `React.StrictMode` in your app like this:

```js
import { StrictMode, Component } from "react";
import { createRoot } from "react-dom/client";

class BlackMamba extends Component {
  componentDidMount() {
    this.refs.venom.focus();
  }

  render() {
    return (
      <>
        <h1>Black Mamba</h1>
        <input ref="venom" />
      </>
    );
  }
}

const root = createRoot(document.getElementById("root"));

root.render(
  <StrictMode>
    <BlackMamba />
  </StrictMode>
);
```

You'll see a warning in the console that looks like this:

>Warning: A string ref, "venom", has been found within a strict mode tree. String refs are a source of potential bugs and should be avoided. We recommend using useRef() or createRef() instead. Learn more about using refs safely here: https://reactjs.org/link/strict-mode-string-ref

![strict-mode-warning](https://res.cloudinary.com/practicaldev/image/fetch/s--rp6j0iUi--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/tnf0qdctk8q7q9d1mzf0.png)

Let's fix it with `createRef` API:

```js
import { StrictMode, Component, createRef } from "react";
import { createRoot } from "react-dom/client";

class BlackMamba extends Component {
  constructor(props) {
    super(props);

    this.venom = createRef();
  }

  componentDidMount() {
    this.venom.current.focus();
  }

  render() {
    return (
      <>
        <h1>Black Mamba</h1>
        <input ref={this.venom} />
      </>
    );
  }
}

const root = createRoot(document.getElementById("root"));

root.render(
  <StrictMode>
    <BlackMamba />
  </StrictMode>
);
```

Bravo, it's fixed :)

![fixed-codesandbox](https://res.cloudinary.com/practicaldev/image/fetch/s--tMTsvnEy--/c_limit%2Cf_auto%2Cfl_progressive%2Cq_auto%2Cw_880/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0pq7w11m292b3sh9e3aj.png)

## Detecting Legacy Context API

The [legacy Context API](https://legacy.reactjs.org/docs/legacy-context.html) is deprecated in React and has been replaced with the new Context API. React Strict Mode can help you detect any usage of the legacy Context API and encourage you to switch to the [new API](https://react.dev/reference/react/useContext).


If you're not already using strict mode, it's definitely worth considering!!!
