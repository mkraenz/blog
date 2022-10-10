---
title: TypeScript's `as` keyword might not be what you think
description: Let's explore the `as` keyword in TypeScript and see how you can use it to stay safe at compile time and runtime.
tags: 'typescript, javascript'
cover_image: typescript-cover_image.png
published: false
id: 702771
---

I've done a lot of interviews on the interviewer-side a little while ago and one thing surprised me:
Many developers get the concept of `as string` in TypeScript wrong; thinking they are save now that they use it - but really aren't. 😕
So what does `as string` or more generally `as MyType` actually do? And how do I fill the gap it leaves?

# tl;dr

`as` keyword only changes _compile_-time behavior. You essentially tell the compiler to stop complaining. Runtime behavior stays unchanged. If you need runtime safety, use techniques like polymorphism, `in` type guards, `typeof` type guards, `instanceof` type guards, or user-defined type guards, or assertion functions.

# Contents

- [What `as string` actually does](#what-as-string-actually-does)
- [Buggy Example](#buggy-example)
- [Polymorphism](#polymorphism)
- [Type Guards](#type-guards)
  - [`in` Type Guard](#in-type-guard)
  - [`instanceof` Type Guard](#instanceof-type-guard)
  - [User-Defined Type Guard](#user-defined-type-guard)
  - [TypeScript trusts us in writing proper User-Defined Type Guards](#typescript-trusts-us-in-writing-proper-user-defined-type-guards)
- [What about `<MyType>`?](#what-about-mytype)
- [What about the new `satisfies`?](#what-about-the-new-satisfies)
- [Closing](#closing)
- [Recommended Reading](#recommended-reading)

# What `as string` actually does

Or: _Type Assertion_ vs _Type Casting_.

Many developers know Type Casting from languages like Java. You have a `double` and want to turn it into an `int`, so you do `int myInt = (int) myDouble;`. In Java this is fine and actually changes the behavior _at runtime_. But here's the thing: **TypeScript doesn't!**

Instead, TypeScript's `as MyType` only changes _compile time_ behavior. It's like saying to the compiler "Compiler, believe me. This is a `MyType`. So stop complaining." Nothing has changed for the resulting JavaScript code after compiling. In other words, we're still completely vulnerable at _runtime_. That's why TypeScript actually uses the term _Type Assertion_ intead of _Type Casting_.

# Buggy Example

Let's look at an example where things go wrong. The code will result in a `addLemon is not a function` runtime error. You can try by hitting the "Run" button inside the

[Interactive TypeScript Playground here](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgCoTsg3gKGcuAE0IBkIBbAexAAoBKALmQDdLhCBuHAXx1EliIUAYUowYEFLnxFCAZQCuAczhR6TVuy68cMBSARhg1AsQCiADzBQ4NABaUwAIQjNocJRCbpMAH2Si4pJ02HjIAPThyABGyshQEAAOlFBgyKAIlOSgSmH2ji5uNp4EAM5oGHQAdLJkVLR02jg4mSClaYhgCnAANj0AnoESXgFiw8gAvKEyxIoqUEz0kwB80-jIraWUPRBVPZRKNABEAILEEITIpcqqR41h3AA0PFyylta2nd19g2PBHEA).

```ts
interface Tea {
  addLemon(): void;
}
interface Coffee {
  addSugar(): void;
}

function addExtra(hotBeverage: Tea | Coffee) {
  // bug report incoming
  (hotBeverage as Tea).addLemon();
}

const actuallyCoffee: Coffee = {
  addSugar: () => {
    console.log("Added sugar");
  },
};
addExtra(actuallyCoffee);
```

Let's walk through this code. We define interfaces `Tea` and `Coffee` with different methods on them. Next, we incidentally grab ourselves a Coffee (what a grave mistake), and pass it to the `addExtra` function. This function however **misuses** `as Tea`, essentially assuming/asserting everything we pass to the function is always tea. The result is a runtime error saying `TypeError: hotBeverage.addLemon is not a function`. Making it into production might easily cost us a few thousand bucks to fix and a lot of customer frustration because they can't drink their morning coffee. At least your customers can still drink `Tea`. 😉

Admittedly, this example seems slightly artificial but there are enough examples in the real world where similar things happen.

# Polymorphism

Depending on what you want from a business perspective, the solution might be using _Type Guards_. Another approach for the above is using polymorphism.

Wikipedia's intro sentence for [Polymorphism](<https://en.wikipedia.org/wiki/Polymorphism_(computer_science)>) is

> In programming languages and type theory, polymorphism is the provision of a single interface to entities of different types [...]

That is, instead of having an `addExtra` function, we might change the code into the following.

[Interactive TypeScript Playground here](https://www.typescriptlang.org/play?#code/FASwdgLgpgTgZgQwMZQAQBUoNQb2K1BAEyIAooAPCGBALlQGdrwBzASnoDcB7EIgbmABfUJFiIUqAMLc4cKGjwFiZStTqNmYdl14DhwYEm5gmhJBACuCADY2AnjLkL6T+WgC8ufIRL1yVDT0TDCsbKgeAHzeBATGptw2UAB0NtwspAAGKlBEqAAkOGo0QplsPkIANMKCyFa2Dm4KySqkAEQMliwIMG1s-EA)

```ts
interface Tea {
  add(extra: string): void;
}
interface Coffee {
  add(extra: string): void;
}

const actuallyCoffee: Coffee = {
  add: (extra: string) => {
    console.log(`added ${extra}`);
  },
};
actuallyCoffee.add("sugar");
```

This also allows adding different things into our tea and coffee. In case, we want to restrict what can be added into the coffee, we can use TypeScript's [Literal Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types).

```ts
interface Coffee {
  add(extra: "sugar"): void;
}
// ...
actuallyCoffee.add("sugar"); // still works
actuallyCoffee.add("lemon"); // compile error: '"lemon"' is not assignable to parameter of type '"sugar"'
```

# Type Guards

Type Guards are similar to `as` in terms of changing the compile type but additionally add a _runtime check_! Note however that since oftentimes we're writing the type guard ourselves, there is still some risk of getting it wrong. So be careful and add unit tests when using more advanced type guards. Let's get started with the most common type guard. The following list is not exhaustive but each one I showcase fixes our problem-case above.

## `in` Type Guard

[Interactive TypeScript Playground here](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgCoTsg3gKGcuAE0IBkIBbAexAAoBKALmQDdLhCBuHAXx1EliIUAYUowYEFLnxFCAZQCuAczhR6TVuy68cMBSARhg1AsQCiADzBQ4NABaUwAIQjNocJRCbpMAH2Si4pJ02HjIwDDINABEsmRUINHhIMgOzq7uniHS+KmOLm42ngB0cRTU9BzIAPTVaHLIIKpQlADuAM7IhG0pYHYoYACeAA4DlGgYYdzIEAA27VJh+GkFmRClxIoqanRVtfWNzW2d3a29-chDo5fjgRIQUzw4OAjU7WAEhgpws7ODd5ImACUABeUIyTbKVRMejIEEAPnBuVeIHalFm61mlCUMVkEEIyHaUKg0V2UwANDwuDhZJZrLZEGBvr9-mJ7rsanVZJ0idtkK1gH1KAoPnAQIMZlAWlB2kA)

```ts
interface Tea {
  addLemon(): void;
}
interface Coffee {
  addSugar(): void;
}

function addExtra(hotBeverage: Tea | Coffee) {
  if ("addLemon" in hotBeverage) {
    hotBeverage.addLemon(); // TS narrows down the type to Tea
  } else {
    hotBeverage.addSugar(); // TS narrows down the type to Coffee
  }
}

const actuallyCoffee: Coffee = {
  addSugar: () => {
    console.log("added sugar");
  },
};

addExtra(actuallyCoffee); // adds sugar without any errors
```

The `in` keyword in JavaScript and TypeScript checks whether a property exists on an object. In the above example, we use it to check whether the `addLemon` property exists on the `hotBeverage` object _at compile time and at runtime_. If it does, TypeScript know it's a `Tea` because the other option `Coffee` does not have an `addLemon` property. On the other hand, if the object does not have an `addLemon` property it must be `Coffee`. Unlike `as`, since `in` is a JavaScript keyword it acts at runtime, too.

## `instanceof` Type Guard

This type guard is only usable if `Tea` or `Coffee` are classes. In the above case, they are only interfaces. Let's pretend for a moment that Tea is a class anyway. In this case, we can use the `instanceof` type guard.

[Interactive TypeScript Playground here](https://www.typescriptlang.org/play?#code/MYGwhgzhAEAqCmZoG8BQ1pgCZYDLwFsB7AOwAoBKFAX1VtQEsSAXeAJwDMxh5oBhIhw7xeaDNiwBlAK4BzMG0oAuaADciDLAG46qVB2klgzBqUw4AogA9mbMGQAWRZgCF4q9mFnwVCJAB9+QWF4KjFoAHoI6CYIZjAjeEFoZgBPAAdeWWkFLHQYjmhHZzcPO28YkjiEnmS-MPyMJ1d3T28AOgl8YnIKLUjo2EloEgU2IgB3GCxJkhSHXjTMlKI4RHzqaHgQCFFG6GbStvhOnBl5RT6BuGHRtnGp6BmJueYFlIzF1YEhEQ3dVDAUhxTDGHIgECpH4hFTQkTQAC8KHyEnOChUlERAD5kRgMECqkQQCcQERZGQAEQSeBYaAQOQKCl9DYAGjoOlQEmstns3GY4MhcNC-Si5iwMHpF2gEwYbyI0mYmBIqS29yIbAgQA)

```ts
class Tea {
  addLemon() {}
}

interface Coffee {
  addSugar(): void;
}

function addExtra(hotBeverage: Tea | Coffee) {
  // instanceof type guard
  if (hotBeverage instanceof Tea) {
    hotBeverage.addLemon(); // TS narrows down the type to Tea
  } else {
    hotBeverage.addSugar(); // TS narrows down the type to Coffee
  }
}

const actuallyCoffee: Coffee = {
  addSugar: () => {
    console.log("added sugar");
  },
};

addExtra(actuallyCoffee); // adds sugar without any errors
```

## User-Defined Type Guard

User-Defined Type Guards allow us to go beyond what TypeScript can infer from `instanceof` and `in` type guards.

[Interactive TS Playground here](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgCoTsg3gKGcuAE0IBkIBbAexAAoBKALmQDdLhCBuHAXx1EliIUAYUowYEFLnxFCAZQCuAczhR6TVuy68cAel3IECgM5hK5ZGACeABxRKFqwjhgKQCMMGrJgx9HBoADyZ-ZAAfZFFxSUZkQJ9jNAxsPGR9SwALaBRVFHI4ECtkAHc4K0SzEqhgSEzfADpkAEkQOsSEOGMIABpm5HgAGwHkACNEAGtLSkyUAANQWctbe0coQnrU4BhkGgAiWTIqEF2fVviAMnOluzE4+oOKbwBeF+Rd13dPal26FPx8KAQMAKKCtMBQBQQLj4XgAoEg1qDLraHAuNweLytWQAUUC4ICGUoYAAQhBmNA4EoICFkhEohIIL9pD5tjRfP4aISSWSKVS6EzUvguaTyVBKRB7sRDtR6Bw0gZUHJkCBVFBKMVEoR1WCstcUJV-KluMgIAMun9-sKeWKqZL5MpVLL5WglSqoGqNcgtcUdfrllNImIGUaeKiENRTAQPI4hlZ6ZImPGUE8LbJFCooEx6MgngA+C34cMgYyUAYSgaUJQ0ADksgghGQxgdUGrdCN3R4XBwOLxYpoiGBcFjSbocvSskSTYzJRqhIUYAIhRN7soUGMQA).

```ts
interface Tea {
  addLemon(): void;
}
interface Coffee {
  addSugar(): void;
}

// custom type guard
function isTea(x: Tea | Coffee): x is Tea {
  if ("addLemon" in x && typeof x.addLemon === "function") {
    return true;
  }
  return false;
}

function addExtra(hotBeverage: Tea | Coffee) {
  if (isTea(hotBeverage)) {
    hotBeverage.addLemon(); // TS narrows down the type to Tea
  } else {
    hotBeverage.addSugar(); // TS narrows down the type to Coffee
  }
}

const actuallyCoffee: Coffee = {
  addSugar: () => {
    console.log("added sugar");
  },
};

addExtra(actuallyCoffee); // adds sugar without any errors
```

The type guard `isTea` is a function that receives a single parameter, returns a boolean, and is marked with `x is Tea`. More generally, it's marked with `myParameter is MyType`. TypeScript will use it to narrow down the type in the `if-statement` of `addExtra`

## TypeScript trusts us in writing proper User-Defined Type Guards

The fact that TypeScript trusts us in implementing a proper user-defined type guard can be shown easily.
Take a look at the following, obviously wrong type guard. TypeScript believes in our abilities and won't budge - the executed JavaScript will.

[Interactive TypeScript Playground here](https://www.typescriptlang.org/play?#code/MYGwhgzhAEAqCmZoG8BQ1pgCZYDLwFsB7AOwAoBKFAX1VtQDMBXE4AFwEtToOIEwyADwBc0FgGsSRAO4kKowTxj8U6aACd4bJupLQ26pvADcdVKmCkIbaAHN1HAF6OQATwBCiddAC8NUxwM0GS8-GT2Ti4eXhRUaBgA9Al2WjDwgvDATGzwWGoRzm6eYOoAdNh4hKSUxtBJ0FLQlgQADhwg8NDw6upE6gA00ABG2ZgaLJwEnd296nRAA)

```ts
function isTea(x: unknown): x is Tea {
  return true;
}

const grizzlyBear = {};
if (isTea(grizzlyBear)) {
  // gets executed
  grizzlyBear.addLemon(); // no compile error, but a runtime error
}
```

This simply says everything is `Tea`. Even though it would be a nice world to live in, that's not how it works in real life. 🙂

# What about `<MyType>`?

You might have seen Type Assertions in the form `const tea = <Tea>{color: 'green'}` before. This is the same as `as Tea`. So again, it doesn't change any runtime behavior, leaving ourselves open for bad surprises.

The `<MyType>` syntax has been deprecated because it becomes confusing in the context of `.jsx` or `.tsx` files as you might encounter using React. Both use the same syntax. Thus, TypeScript recommends always using `as MyType`.

> TypeScript recommends always using `as MyType` instead of `<MyType>`.

To clarify, take a look at this.

```tsx
const beverage = <Tea>{ color: 'green' };
return <Tea>{beverage.color}</Tea>
```

Here we have a _component_ named `Tea` but also a variable `beverage` with a type assertion of _type_ `Tea`. If you are confused now, that's intended. 🙃 With `as Tea` it's slightly easier to identify whether we talk about a component or a type:

```ts
const beverage = { color: 'green' } as Tea;
return <Tea>{beverage.color}</Tea>
```

# What about the new `satisfies`?

The upcoming [TypeScript 4.9](https://devblogs.microsoft.com/typescript/announcing-typescript-4-9-beta/#hamilton) brings in the new keyword `satisfies`. Functionally, it acts very similar to `as` with the additional benefit of not broadening the type to something more generic. For us, the important lesson here is that `satisfies` leaves us _just as open to runtime errors_ as `as` does. So be careful!

# Closing

Wanna learn more about TypeScript? Then hit subscribe on the blog, and join me on [Twitch TypeScriptTeatime](https://www.twitch.tv/typescriptteatime)! Looking forward to see you!

# Recommended Reading

- [basarat's TypeScript book - Type Assertions](https://basarat.gitbook.io/typescript/type-system/type-assertion)
- [TypeScript docs - Type Guards](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-guards-and-differentiating-types) (deprecated page but there's no official replacement about Type Guards at time of writing)
- [TypeScript docs - instanceof Type Guards](https://www.typescriptlang.org/docs/handbook/advanced-types.html#instanceof-type-guards)
