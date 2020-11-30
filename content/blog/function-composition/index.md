---
title: Function Composition
date: "2020-10-28"
description: "What is function composition and how do we do it?"
canonical: "https://www.aaron-thompson.dev/function-composition"
---

All of software development is composition. The breaking down of huge problems into smaller pieces and stitching them together so that, not only are the smaller problems solved independently of each other but they work together to solve the bigger problem. JavaScript Applications mix functional programming and object-oriented programming extensively. We make objects for our functions and functions can make more objects.

The breaking down of problems into smaller problems and stitching them together (from a functional perspective) is the topic of this post. How you do it matters for your growing codebase.

## Composing Functions

What does it mean to 'compose' functions? Take a look at the below example:

```jsx
import { doSomething, doSomethingElse, createFinalData } from "./functions"

// Composing example 1
const doALot = data => {
  const editedData = doSomething(data)
  const furtherEditedData = doSomethingElse(editedData)
  const finalData = createFinalData(furtherEditedData)
  return finalData
}

const result = doALot(someData)

// Composing example 2
const pipe = (...fns) => x => fns.reduce((y, f) => f(y), x)

const doALot = pipe(doSomething, doSomethingElse, createFinalData)

const result = doALot(someData)
```

Before we move on, let's breakdown that `pipe` function as I found it hard to understand at first.

1. `pipe` takes a list of as many functions as you like and gathers them together into an array called `fns`. That's what the rest (`...`) operator is doing.
2. Passing a list of functions returns another function that takes an initial value (which above is `someData`). `fns` is then iterated over using `[.reduce](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)` and each function is applied to the output of the one before it.

As you can see from the two examples, in both cases we are passing data down a 'pipeline' to be transformed. Even if perhaps a little unfamiliar, the second example is _objectively_ better because we eliminate wiring and connectivity code. `pipe` implicitly passes the return value to the next function and reduces the surface area of our code. Less surface area means less chance for bugs, less syntactical 'noise', and less cognitive load when reading.

The implementation details of the functions in the pipeline above are hidden for the simplicity of the example, but you can see that they all take one input and return one output. This output is then passed to the next in line. What if `doSomething` needed 2 parameters? Can we still use `pipe` like above? Yep! Keep reading.

To learn how to compose functions like this we need to understand currying.

## Currying

Normally a function might expect multiple parameters like this:

```jsx
const add3 = (a, b, c) => a + b + c

add3(1, 2, 3) // 6
```

A curried version of this function would look like this:

```jsx
const add3 = a => b => c => a + b + c

add3(1)(2)(3) //6
```

You can see here that a curried function takes its arguments one at a time but ends up with the same result.

The reason that curried functions are so convenient is that they transform functions that expect multiple parameters into functions that accept one arg at a time. This means that they can fit into functions composition pipelines like `pipe` in our example above.

A function can take any number of inputs, but can only return a single output. For functions to be composable, the output type must align with the expected input type:

```jsx
f: a => b
g: b => c
```

What if `g` was expecting two inputs?

```jsx
f: a => b
g: (x, b) => c
```

How would we introduce `x`? We would have to curry `g`. Going back to our `doSomething` example. If it in fact, required 2 parameters, we could define it something like this:

```jsx
const doSomething = data1 => data2 => {
  /* something */
}
```

Note that it is not enough to simply curry. You need to make sure the functions take the parameters in the right order. This is often called the data-last approach whereby the application is gradually 'specialized' and the data is passed last to provide the result.

What do you think of function composition like this? Let me know in the comments.

**References:**

1. [https://medium.com/javascript-scene/master-the-javascript-interview-what-is-function-composition-20dfb109a1a0](https://medium.com/javascript-scene/master-the-javascript-interview-what-is-function-composition-20dfb109a1a0)
2. [https://medium.com/javascript-scene/curry-and-function-composition-2c208d774983](https://medium.com/javascript-scene/curry-and-function-composition-2c208d774983)
