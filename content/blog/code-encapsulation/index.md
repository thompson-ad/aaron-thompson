---
title: Code Encapsulation
date: "2020-10-25"
description: "Timeless principles of software engineering"
canonical: "https://www.aaron-thompson.dev/code-encapsulation"
---

Recently I've been trying to become a better software engineer and programmer and something I’m trying to get my head around is how to 'encapsulate' your code well. Encapsulation produces code that has loose coupling but high cohesion. Capsules of your code work seamlessly together but also independently of each other.

As put by Eric Elliott [here](https://medium.com/javascript-scene/encapsulation-in-javascript-26be60e325b4), the encapsulation of code is the bundling of data and the methods that act on that data such that access to that data is restricted from outside the bundle. It's the local retention, hiding, and protection of state processes.

Code that is well encapsulated keeps in mind three timeless principles in software engineering:

- **Avoid shared mutable state. “Nondeterminism = parallel processing + mutable state”** — Martin Odersky, designer of the Scala programming language
- **“Program to an interface, not an implementation.”** — Gang of Four, “Design Patterns: Elements of Reusable Object-Oriented Software”
- **“A small change in requirements should necessitate a correspondingly small change in the software.”** — N. D. Birrell, M. A. Ould, “A Practical Handbook for Software Development”

I'm still getting to grips with what these quotes truly mean and practicing them in my work but let's outline each in turn briefly to try shed some light:

First off, shared mutable state. This is where different parts of your code depend on the same data and that data is permanently modified by these parts. The input of one thing might depend on some state that is also modified by something else. If your program decides to run in a different order or parts run at the same time, chaos ensues! The results are unpredictable. Sometimes it works and sometimes it doesn’t.

Secondly, programming to an interface. This, from what I understand is programming by message passing. Message passing means that, instead of updating an object's properties directly, you call one of its methods and it *might* do what you want. This idea of encapsulating your code behind a public interface is interesting because it also addresses the third point above: "A small change in requirements should necessitate a correspondingly small change in the software". When you program like this it means that other code is not linked to implementation details. It just knows what message to pass.

Ok so we sort of know what encapsulation means but what does it look like in JS. Let's see a simple example:

**Factory functions** + **Closures**

In this example, the `accountBalance` is encapsulated within the `createPerson` factory function and can only be manipulated by calling `pay()` and `getBalance()` .

These are privileged methods which means they have access to the private data inside the containing function's scope, even after the function has returned. The references are also live, meaning that if the `accountBalance` changes it will change for every privileged function with access to it.

```jsx
const createPerson = ({ name = "thing", age = 20 } = {}) => {
  let accountBalance = 10

  return {
    pay: () => accountBalance++,
    getBalance: () => accountBalance.toLocaleString(),
  }
}

const person = createPerson()
person.pay()
console.log(person.getBalance()) // '11'
```

This is a very simple example of such an important concept but it highlights how certain data and code can be hidden behind a public interface and can only be manipulated by passing messages to the object instance created.

_References:_

1. [https://medium.com/javascript-scene/encapsulation-in-javascript-26be60e325b4](https://medium.com/javascript-scene/encapsulation-in-javascript-26be60e325b4)
