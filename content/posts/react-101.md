---
date: "2023-02-01"
title: "React 101"
tags: ["react", javascript", "ucalgary"]
author: "masoudkf"
description: "React basics. This post is part of the course material for [ENSF 381 (W23)](https://contacts.ucalgary.ca/info/enel/courses/w23/ENSF381?destination=courses%2Fw23) at the University of Calgary."
---

## Overview 

React is a JavaScript library built by Facebook (now Meta) for creating user interfaces. Mainly, there are two advantages of using React:

- We can update the DOM using a declarative approach (as opposed to imperative)
- We can break down the UI into reusable components

Declarative means we only need to be concerned about the states of our application (the data we manage). The rest (updating the DOM and refreshing the page) will be React's problem to solve (and it can solve it pretty efficiently).

React is component-based; that is, we can write reusable components (in separate files) and then use them to build the UI. This helps the project to be more organized and structured. It also keeps our code DRY (Don't Repeat Yourself).

### Creating a React Application

There are different ways of creating and starting with a React application. Here, we're going to use a `Node.js` application called `create-react-app`. Although you don't need to use this application, it comes with many benefits (from React website):

- Scaling to many files and components
- Using third-party libraries from `npm`
- Detecting common mistakes early
- Live-editing CSS and JS in development
- Optimizing the output for production

#### Using `create-react-app`
Before using `create-react-app`, make sure you have `node`, `npm`, and `npx` already installed on your system. You can check by using the following commands. If you get a version back and not `command not found`, then you're good to go.

```bash
node --version
```

```bash
npm --version
```

```bash
npx --version
```

If all is good, you can use this command in your terminal to create a new React application:

```bash
npx create-react-app my-new-app
```

If successful, you will have a new folder named `my-new-app` in the directory you ran the command in. It has everything you need to start a React application.

### JSX

You combine HTML tags and JavaScript and you get yourself JSX. That's basically it. Here's a quick example:

```jsx
const name = "Josh Perez";
const element = <h1>Hello, {name}</h1>;
```

Browsers don't support JSX right out of the box, so we need libraries to compile JSX into plain JavaScript so that browsers can understand it. But don't worry. You don't need to be concerned about those libraries. The  `create-react-app` application comes with many pre-installed libraries including JSX parsers. 

### Components

React uses components. Basically, you break your application (and by application I mean the front-end of your application, since, as I mentioned at the beginning, React is a JavaScript framework for building user interfaces). It's always a good idea to spend some good time designing your application and breaking it into different components before writing your React code. 

There are many ways to build a user interface with React. In one of the previous semesters, we gave the students the same project but they all submitted a completely different design at the end. Not 2 applications were the same. That's the beauty of React; you can build your application in many ways. However, not every design is good and efficient. I always say that the more time you spend in the designing and analyzing phase of your application, the more efficient your final application will be; the performance will be better and the whole process will go much smoother. If you jump into the editor and start coding right away, you might think that you are saving time, but in big projects, you will see how important the designing and analyzing phase is.

You can consider Components as classes. In fact, in previous versions of React, the components were all JavaScript classes. You can still use classes for writing React applications, but it is considered a past approach. The newer approach is using functions. If you hear somewhere about Functional React, that's just React using functions as components instead of classes.

But it's safe to still think of components as classes. I assume that you have already passed some programming courses and are familiar with Object Oriented Programming. An application (let's say a Java application), is usually consisted of multiple classes. And these classes are talking to each other. So, there's always a way for classes to communicate with one another. Components in React are not an exception. React Components, too, need to talk to each other, and the way we accomplish that is with a concept called Props (short for Properties). So, basically, if you want to send something to another Component, you will do so by putting your stuff in Props and then passing the Props to the target class and vice versa.

Here's an example of passing Props to another component:

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
```

### States

States are what keep your React application alive. And by alive, I mean an application that reacts (pun intended) to users' actions. States are different from normal JavaScript variables. For one, we don't update React states directly by the equal sign (`=`). There are functions coming from the React library that take care of that for us.

But how we can decide between creating a JavaScript variable or a React state? So, basically, if you have a variable that meets the following conditions, you need to create a React state for it:

- Its value changes over time (without refreshing your browser)
- Its value has an impact on the UI (it changes something in the UI)

So, with that definition, if I have a key that I use for calling an API endpoint, I don't need to create a React state for that. It never changes during the lifetime of a webpage. But, if I have a number showing the total number of users registered in my application and the number changes asynchronously (without refreshing the webpage), I should create a React state for that. Why? Because

1. It changes
2. Its change affects the UI (you're watching the page and then the number goes from 120 to 125. That's a visible change in the UI. You can see it!)

### Hooks

When it was just classes in React, there was no Hook. Hooks are a newish addition to React and if you want to write functional React (which we want in this course), you need to learn them. Basically, they are a way to deal with the states of your applications (defining them, changing them, handling them) without using classes.

For example, we use `useState` to create React states. Take a look at the following example:

```jsx
import React, { useState } from "react";

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

`useState` accepts one argument which is the default value of the state being declared (`count` in the above example), and returns an array of two:

1. The state itself
2. A function to change the value of the state

The above example uses JavaScript destructuring to get the two values and put them in two variables: `count` and `setCount`. We talked about the JavaScript destructuring capability in previous chapters.

In the workshops, especially the longer one, I talked about another useful hook that we can use to change a React state. That is the `useReducer` hook. So, basically, if you have a complicated logic to change your state, you might be better off using `useReducer`. But if you have a pretty simple logic, like if I click on this button, increase the value of a counter by one, then using `useState` is enough, although nothing stops you from using `useReducer` even in this situation.

Here's an example of using `useReducer`:

```jsx
const initialState = { count: 0 };

function reducer(state, action) {
  switch (action.type) {
    case "increment":
      return { count: state.count + 1 };
    case "decrement":
      return { count: state.count - 1 };
    default:
      throw new Error();
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: "decrement" })}>-</button>
      <button onClick={() => dispatch({ type: "increment" })}>+</button>
    </>
  );
}
```

---

### React Router

React Router gives you the ability to simulate URL navigations by loading/unloading React components. For example, if the URL is `/about`, load (show) the `About` component; if it's `/skills`, unload the `About` component and load the `Skills` component instead.

In order to use React Router, you first need to install it using `npm` as it doesn't come pre-installed with `create-react-app`. Make sure you're at the root of your React project and then run:
```bash
npm install react-router-dom
```

You can now use React Router in your project. Create a component named `About` and another one named `Skills`:

`src/About.js`:
```javascript
function About() {
  return (
    <h1>About</h1>
  )
};

export default About;
```

`src/Skills.js`:
```javascript
function Skills() {
  return (
    <h1>Skills</h1>
  )
};

export default Skills;
```

Then in your `App.js` file:

`src/App.js`:
```javascript
import { BrowserRouter, Routes, Route } from "react-router-dom";
import About from "./About";
import Skills from "./Skills";

function App() {
  return(
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<About />}></Route>
        <Route path="/skills" element={<Skills />}></Route>
      </Routes>
    </BrowserRouter>
  )
}

export default App;
```

We declared two routes:
- `/` which loads the `About` component
- `/skills` which loads the `Skills` component

Run the application. It should show the `About` component. Add `/skills` to the end of the URL and you should see the `Skills` component instead.
