---
title: Closure and Common Use Cases
date: "2020-10-28"
description: "Explaining closure through common use cases"
canonical: "https://www.aaron-thompson.dev/closure-and-common-use-cases"
---

> A closure is a function bundled with its lexical environment

JavaScript is a lexically scoped language. This means that functions use the variable scope that was in effect when they were **defined** (**not** the variable scope in effect when they are **invoked**).

Technically, all JavaScript functions are closures, but because most functions are invoked from the same scope they were defined, it doesn't matter that there was a closure involved.

Closures are commonly used for [encapsulation](https://www.aaron-thompson.dev/posts/code-encapsulation-40mm/) (the ability to have private properties for objects), functional programming (curried functions, partial applications) and to grant access to variables inside event listeners.

Let's take a look at each of these use cases to help us understand what closure is.

## Encapsulation

Say we had a [factory function](https://www.aaron-thompson.dev/posts/objects-and-how-to-make-them-26n7/) that returned a counter object:

```jsx
const counter = () => ({
  n: 0,
  count() {
    this.n++
  },
  reset() {
    this.n = 0
  },
})

const counter1 = counter()
counter1.count()
counter1.count()
console.log(counter1.n) // 2
counter1.n = 0 // << We don't want this
console.log(counter1) // { n: 0, ... } uh oh!
```

Buggy or malicious code could reset the counter without calling the `reset()` method as shown above.

As mentioned in my post on [encapsulation](https://www.aaron-thompson.dev/posts/code-encapsulation-40mm/), this breaks a fundamental principle of good software design:

> “Program to an interface, not an implementation.” — Gang of Four, “Design Patterns: Elements of Reusable Object-Oriented Software”

We only want to be able to communicate with `counter` by using its interface and by passing messages (methods) such as `count()` or `reset()`. We don't want to be able to reach in and manipulate properties such as `n` directly. Unfortunately, the property `n` forms part of the public interface for this object and so it is easily manipulated. Let's change that. Closure can help us out here. Take a look at this revised example:

```jsx
const counter = () => {
  let n = 0
  return {
    count() {
      n++
    },
    reset() {
      n = 0
    },
    getCount() {
      console.log(n)
    },
  }
}

const counter1 = counter()
counter1.count()
counter1.count()
counter1.getCount() // 2
console.log(counter1.n) // undefined
```

Before we dissect this. Reconsider our definition of closure - a function bundled with its lexical environment. The lexical environment being the variable scope that was in effect when the function was defined.

`n` is in scope when `count`, `reset` and `getCount` are defined and so, when counter returns and the object is created, the only code that will have direct access to `n` is this instance of the counter object and the methods on it.

Note that the reference to `n` is live and each invocation of counter creates a new scope independent of scopes created by previous invocations and a new private variable within that scope. So what is `n` for `counter1` may not the what is `n` for `counter2`.

## Partial application

A partial application is a function that has been applied some but not all of it's arguments. Let's look at an example:

```jsx
const trace = label => value => {
  console.log(`${label}: ${value}`)
}
```

`trace` is a function that takes a label and a value and logs it to the console.

Because this function is curried we can create specialist 'sub-functions' that are partial applications of the full trace function:

```jsx
const traceLabelX = trace("Label X")

console.log(traceLabelX.toString()) // 'value => {console.log(`${label}: ${value}`);}'

traceLabelX(20) // 'Label X : 20'
```

If you log `traceLabelX` to the console you see it return a function that takes in a value and logs the label and the value. But where is `label`? This function's closure has access to the `label` it was returned with anywhere it is now used.

## Event listeners

Open up VSCode and make this little `.html` page and open it up in a browser.

```jsx
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    Closures in event listeners
  </body>

  <script>
    const body = document.body;
    const initButtons = () => {
      let button;
      for (var i = 0; i < 5; i++) {
        button = document.createElement("button");
        button.innerHTML = "Button " + i;
        button.addEventListener("click", (e) => {
          alert(i);
        });
        body.appendChild(button);
      }
    };
    initButtons();
  </script>
</html>
```

What do you think happens when you click on the buttons? Each button click will return an alert with '5'. Why is this? The first thing to note here is that we are using `var` not `let` to declare `i`. As such this is a bit of a contrived example as you would very rarely use `var` for variable declaration these days but stick with me as it will help you understand closures. Remember - `var` is **function** scoped and `let` is **block** scoped.

The `for` loop is within the `initButtons` function and `var` is 'hoisted' to the top of the function.

Every time a loop completes a button is created with an attached event listener who's callback has reference to `i`. As subsequent loops complete, `i` updates, as too does each event-listeners reference to it. This is the problem, every closure has access to the same reference to `i`.

We could fix this in a couple of ways:

```jsx
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    Closures in event listeners
  </body>

  <script>
    const body = document.body;

    const initButton = (name, alertMessage) => {
      button = document.createElement("button");
      button.innerHTML = "Button " + name;
      button.addEventListener("click", (e) => {
        alert(alertMessage);
      });
      body.appendChild(button);
    };

    for (var i = 0; i < 5; i++) {
      initButton(i, i);
    }
  </script>
</html>
```

Each event listener is now scoped to the `alertMessage` param which is defined on function invocation.

```jsx
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    Closures in event listeners
  </body>

  <script>
    const body = document.body;

    const initButtons = () => {
      let button;

      for (let i = 0; i < 5; i++) {
        button = document.createElement("button");
        button.innerHTML = "Button " + i;
        button.addEventListener("click", (e) => {
          alert(i);
        });
        body.appendChild(button);
      }
    };
    initButtons();
  </script>
</html>
```

Or just use `let` instead of `var` within the loop. Using `let` will ensure that each iteration of the scope has its own independent binding of `i`.

Has this helped you understand closure? Let me know in the comments!

### References

1. [https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36#.11d4u33p7](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-a-closure-b2f0d2152b36#.11d4u33p7)
2. [https://medium.com/javascript-scene/curry-and-function-composition-2c208d774983](https://medium.com/javascript-scene/curry-and-function-composition-2c208d774983)
3. JavaScript: The Definitive Guide, 7th Edition by David Flanagan
