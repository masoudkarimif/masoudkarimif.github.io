---
date: "2022-12-14"
title: "JavaScript 101"
tags: ["javascript", "ucalgary"]
author: "masoudkf"
description: "JavaScript basics. This post is part of the course material for [ENSF 381 (W23)](https://contacts.ucalgary.ca/info/enel/courses/w23/ENSF381?destination=courses%2Fw23) at the University of Calgary."
---

## The Basics

### Case-sensitive Language
Every identifier in JavaScript is case-sensitive, meaning that a variable with the name `Foo` is different from a variable with the name `foo`:

```javascript
let foo = "foo";
console.log(foo); // "foo"
console.log(Foo); // "Uncaught ReferenceError: Foo is not defined"
```

### Comments
You can use `//` for single-line comments and `/* */` for multi-line comments. Comments are not executed and only used for clarifications:

```javascript
// this has no effect on the execution

/*
This is a multi-line
comment. Again, no effect on the execution
*/
```

### Ending Statements
Although not necessary, it's a good idea to always end your statements with a `;`:

```javascript
let university = "ucalgary";
```

### Dynamiclly Typed
JavaScript is not statically typed, meaning that the true type of a variable is decided in run-time and can change during the execution of the code:

```javascript
let name = "Alice";
console.log(name); // "Alice"

name = 5; // this is fine
console.log(name); // 5
console.log(name + 10); // 15
```

---

## Declaring Variables

### The `const` Keyword

A constant is a "variable" that--unlike variables (declared with the `let` or `var` keywords)--cannot be overwritten. Once declared, you cannot change its value:

```javascript
var college = "bow valley";
const university = "ucalgary";
college = "sait"; // allowed
university = "ubc"; // results in "Uncaught TypeError: Assignment to constant variable." error
```

### The `let` Keyword
The `let` keyword declares a variable just like the `var` keyword does; except that the `let` keyword blocks off the scope of a variable in any `{}` blocks, such as `for` and `if/else`. 

```javascript
var university = "ubc";
if (university == "ubc") {
	var university = "ucalgary";
    console.log(university); // ucalgary
}
console.log(university); // ucalgary
```

```javascript
var university = "ubc";
if (university == "ubc") {
	let university = "ucalgary";
    console.log(university); // ucalgary
}
console.log(university); // ubc
```

---

## Template Strings
Template strings give us a better way to interpolate variables into strings by wrapping them inside `${}`:

```javascript
// old way
console.log(lastName + ", " + firstName + " " + middleName);

// new way
console.log(`${lastName}, ${firstName} ${middleName}`);
```

They also respect whitespace, making it easy to draw up HTML code.

```javascript
document.body.innerHTML = `
<section>
  <header>
      <h1>The UCalgary Blog</h1>
  </header>
  <article>
      <h2>${article.title}</h2>
      ${article.body}
  </article>
  <footer>
      <p>copyright | The UCalgary Blog</p>
  </footer>
</section>
`;
```

---

## Functions
There are multiple ways to create a function in JavaScript. We'll touch on different methods here.

### Function Declarations
Function declarations start with the `function` keyword, folowed by the name of the function, the parameters it receives, an the body wrapped in curly braces `{}`.

```javascript
function greetings(name) {
    return `Greetings, ${name}!`;
}
```

Once declared, you can call it using the function name and the necessary arguments:

```javascript
console.log(greetings("Alice")); // Greetings, Alice!
```

### Function Expressions
A function expression is just a function declaration assigned to a variable. As functions are first-class citizens in JavaScript, you can assign them to a variable, or even pass them around like a normal variable:

```javascript
const greetings = function(name) {
    return `Greetings, ${name}!`;
}

console.log(greetings("Alice")); // Greetings, Alice!
```

One difference between function expressions and declarations is that you can't invoke a function before writing a function expression, but you can do so with function declarations:

```javascript
console.log(greetings("Alice")); // Error: "Uncaught ReferenceError: Cannot access 'greetings' before initialization"

const greetings = function(name) {
    return `Greetings, ${name}!`;
}
```

```javascript
console.log(greetings("Alice")); // Greetings, Alice!

function greetings(name) {
    return `Greetings, ${name}!`;
}
```

### Default Parameters
You can specify a default value for function parameters, just as you would in a language like Python:

```javascript
const greetings = function(name="Unknown Person") {
    return `Greetings, ${name}!`;
}

console.log(greetings()); // Greetings, Unknown Person!
```

### Arrow Functions
Arrow functions are a relatively new addition to JavaScript. You can declare functions without using the `functions` keyword, and sometimes, without having to use the word `return`:

```javascript
const greetings = name => `Greetings, ${name}!`;

console.log(greetings("Alice")); // Greetings, Alice!
```

If the function takes more than one argument, you need to use `()`:

```javascript
const greetings = (name, lastName) => `Greetings, ${name} ${lastName}!`;

console.log(greetings("Alice", "Smith")); // Greetings, Alice Smith!
```

In case you want to return more than one statement, you need to wrap the function in `{}` and use the `return` keyword:

```javascript
const greetings = (name, lastName) => {
  if (name === "Alice") {
    return `Greetings, ${name} ${lastName}! How was the Wonderland?`;
  }

  return `Greetings, ${name} ${lastName}!`;
}

console.log(greetings("Alice", "Smith")); // Greetings, Alice Smith! How was the Wonderland?
```

---

## Objects and Arrays
Objects and Arrays are variables that can contain many values instead of just one. Objects are a container for key/value pairs. In order to access a field inside an object, we use their key:

```javascript
const myObject = {
    name: "Alice",
    occupation: "Student"
};
console.log(myObject.name); // Alice
console.log(myObject.occupation); // Student
```

Arrays are containers for a list of values. In JavaScript, the values don't have to be of the same type. To access an element inside an array, we use the array index, starting from `0`:

```javascript
const frontendTech = ["JavaScript", "HTML", "CSS"];
console.log(frontendTech[1]); // HTML
```

### Destructuring Objects
Destructuring objects allows us to retrieve only the values we're interested in, instead of the whole object:

```javascript
person = {
    name: "Alice",
    occupation: "Student"
};

const { name } = person;
console.log(name); // Alice
```

```javascript
person = {
    name: "Alice",
    occupation: "Student",
    age: 25,
    goesTo: "University of Calgary",
    likes: "Programming"
};

const { name, goesTo, likes } = person;
console.log(`${name} goes to ${goesTo} and likes ${likes}.`); // Alice goes to University of Calgary and likes Programming.
```

We can use destructuring in function parameters too:

```javascript
person = {
    name: "Alice",
    occupation: "Student",
    age: 25,
    goesTo: "University of Calgary",
    likes: "Programming"
};

const printProfile = ({ name, goesTo, likes }) => `${name} goes to ${goesTo} and likes ${likes}.`;
console.log(printProfile(person)); // Alice goes to University of Calgary and likes Programming.
```

It also works for nested types:

```javascript
person = {
    name: "Alice",
    occupation: "Student",
    age: 25,
    goesTo: {
        universityName: "UCalgary",
        universityProvince: "Alberta"
    },
    likes: "Programming"
};

const printProfile = ({ name, goesTo: {universityName, universityProvince}, likes }) => `${name} goes to ${universityName} in the province of ${universityProvince} and likes ${likes}.`;
console.log(printProfile(person)); // Alice goes to UCalgary in the province of Alberta and likes Programming.
```

### Destructuring Arrays
We can also destructure arrays based on their index:

```javascript
const frontendTech = ["JavaScript", "HTML", "CSS"];
const [firstTech] = frontendTech;
console.log(firstTech); // JavaScript

// note how we ignore the first two using ','
const [,,lastTech] = frontendTech;
console.log(lastTech); // CSS
```

---

## Object Enhancements
This is the opposite of destructuring. Basicaly, we structure or create new objects this way:

```javascript
const name = "Alice";
const occupation = "Student";

const enhanced = { name, occupation };
console.log(enhanced.name, enhanced.occupation); // Alice, Student
console.log(enhanced);
```

We can also attach functions to an object:

```javascript
const name = "Alice";
const occupation = "Student";
const printProfile = function () {
  return `Name is ${this.name} and occupation is ${this.occupation}`;
};

const enhanced = { name, occupation, printProfile };
console.log(enhanced.printProfile()); // Name is Alice and occupation is Studentf
```

Note the use of `this` in the `printProfile` function. `this` refers to the object that called the function; in this case: `enhanced`.

---

## The Spread Operator
The spread operator (`...`) allows us to break down (spread) contents of an array or object.

```javascript
const frontendTech = ["JavaScript", "HTML", "CSS"];
console.log(...frontendTech); // "JavaScript", "HTML", "CSS"
```

Let's copy the `frontendTech` array to a new one using the spread operator:

```javascript
const frontendTech = ["JavaScript", "HTML", "CSS"];
const copy = [...frontendTech];
copy[0] = "TypeScript";

const [first] = frontendTech;
const [copyFirst] = copy;
console.log(first); // "JavaScript"
console.log(copyFirst); // "TypeScript"

const shallowCopy = frontendTech;
shallowCopy[0] = "TypeScript";
console.log(frontendTech[0], shallowCopy[0]); // "TypeScript", "TypeScript"
```

The above example shows how we can deep copy an array using the spread operator and change it later without impacting the main array. Otherwise (as shown in the second part of the example), we'll be doing a shallow copy (reference copy) and every change to either arrays will impact the other one as well.

Let's add a new element to an array:

```javascript
const arr = ["Python", "Golang", "Java"];
const arr2 = [...arr, "JavaScript"];
console.log(arr.length, arr2.length); // 3, 4
console.log(arr); // ["Python", "Golang", "Java"]
console.log(arr2); // ["Python", "Golang", "Java", "JavaScript"] 
```

The spread operator also works with objects:

```javascript
const person = {
    name: "Alice",
    occupation: "Student",
    age: 25,
    goesTo: {
        universityName: "UCalgary",
        universityProvince: "Alberta"
    },
    likes: "Programming",
    printLikes () { return `${this.name} likes ${this.likes}` }
};
console.log(person.printLikes()); // "Alice likes Programming"

const updatedPerson = {
    ...person,
    // keep everything as is, but replace the "likes" field
    likes: "Hiking"
};
console.log(updatedPerson.printLikes()); // "Alice likes hiking"
```

---

## Asynchronous JavaScript
Asynchronous programming refers to being able to do something else while waiting for an I/O operation, such as network request, reading files, accessing GPU, etc. As JavaScript is single threaded, asynchronous programming allows us to send network requests without blocking the thread until the request returns. Otherwise, the main thread will be blocked resulting in a browser freeze. 

Promises are the core part of asynchronous JavaScript. They allow us to send a network request, and instead of waiting for the response, JavaScript will give us a Promise right away, which we can later use for gathering the result.

### The `fetch` Function
The `fetch` function is used to asynchronously calling an API endpoint:

```javascript
// the url returns a random fox image
console.log(fetch("https://randomfox.ca/floof/")); // [object Promise] { ... }
```

Instead of getting the result of the API call, we received a Promise. The promise is an object that represents whether the async operation is pending, has been completed, or has failed. Basically, the browser will let us know when the result is ready.

The pending promise represents a state before the data has been fetched. We need to chain on a function called `then`. This function will take in a callback function that will run if the previous operation was successful. In other words, fetch some data, then do something else.

```javascript
fetch("https://randomfox.ca/floof/")
  .then(res => res.json())
  .then(json => json.image)
  .then(console.log) // https://randomfox.ca/images/73.jpg
  .catch(console.error);
```

First, we send a `GET` request to receive a random fox image. Then we use `then` to wait for the response. Whenever the response is ready, the callback function (`res => res.json()`) will be called, which will convert the result to a `JSON` format. Whatever the function returns--in this case a `JSON` object--will be the input for the next `then` function. We gather the `image` field from the result and pass it to `console.log`. If the request fails, the function inside `catch` will be called. Example of a result from the endpoint:

```json
{"image":"https:\/\/randomfox.ca\/images\/44.jpg","link":"https:\/\/randomfox.ca\/?i=44"}
```

### Async/Await
Another way to deal with Asynchronous JavaScript is to use `async/await`. `async` refers to the function that does something asynchronously, such as sending a network request; and `await` is for waiting for the response to come back. Some developers prefer this approach as it reads a little bit more easily that a chain of `then` methods. 

Let's convert the previous example to `async/await`:

```javascript
const getFox = async () => {
    let res = await fetch("https://randomfox.ca/floof/");
    let json = await res.json();
    let { image } = json;

    console.log(image);
}

getFox(); // https://randomfox.ca/images/76.jpg
```

---

## Usefule Array Methods

### Map

The `map()` method creates a new array populated with the results of calling a provided function on every element in the calling array. `map` runs a function for every element inside an array and returns a new array.

```javascript
let articles = [
    {
        name: "Matrix 4 is Trash",
    },
    {
        name: "Scientists Find Out Birds Can Fly",
    },
];

articles = articles.map((article) => ({...article, markup: `<h1>${article.name}</h1>`}));
console.log(articles[0].name, articles[0].markup); // "Matrix 4 is Trash", "<h1>Matrix 4 is Trash</h1>"
```

### Filter

The `filter()` method creates a new array with all the elements that pass the test implemented by the provided function.

```javascript
const languages = [
    "JavaScript",
    "Python",
    "Go",
    "Rust",
    "Java"
];

const result = languages.filter((language) => language.length < 5);
console.log(result); // ["Go","Rust","Java"]
```

---

### localStorage
The `Window` object has a storage API we can use to store some data on the current domain (address). The data will persist until it's deleted, but will only be available on the browser and the device that were used to store the data. For instance, if you use the `localStorage` to persist some data on Chrome, it won't be available when you access the same address on Firefox, or on Chrome on a different device. It also won't be available in private mode. That's why ultimately, applications need to have a central backend to store the data across browsers and devices.

With JavaScript, we can store and then retrieve data to and from `localStorage` like this:

```js
// store some data with the key 'name'
localStorage.setItem('name', 'alice');

// retrieve the data with the same key
const name = localStorage.getItem('name');
console.log(name); // alice
```

More often than not, though, you want to store data with some structure. The most common structure when we work with JavaScript is JSON. In order to store and retrieve data in JSON format, we need to use the `JSON.stringify(data)` and `JSON.parse(data)` functions. Example:

```js
// use stringify when storing structured data
const data = {name: "Alice"};
localStorage.setItem('somekey', JSON.stringify(data));

// use parse when reading the data
const item = JSON.parse(localStorage.getItem('somekey'));
console.log(item.name); // Alice
```

---

## The DOM API
The JavaScript DOM API is the way to interact with the--surprise, surprise--DOM. We use the DOM API to:
- Change the attributes of HTML elements
- Add/Remove new HTML elements
- Add/Remove events, such as mouse click or key down
- ...

Examples:

```javascript
// get an element with a specific id
document.getElementByID("element-id")

// get an array of elements with a specific class 
document.getElementsByClassName("element-class");

// get the first element matching a specific selector
document.querySelector(".test-class");

// create new elements
document.createElement("p");
document.createElement("section");

// set attributes
const img = document.getElementById("image")
img.setAttribute('src', 'new-image-src')

// add element
const paragraph = document.createElement("p")
const content = document.createTextNode("Hello!");
paragraph.appendChild(content)

const div = document.getElementById("wrapper")
div.appendChild(paragraph)

// remove element
const element = document.getElementById("removable");
element.remove();

// add events
const button = document.getElementById("submit")
const alarm = () => alert("Clicked")
button.addEventListener("click", alarm)

// toggle a class
element.classList.toggle("class-name");
// add a class
element.classList.add("class-name");
// remove a class
element.classList.remove("class-name");
```
