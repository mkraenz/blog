---
title: TypeScript's `as` keyword might not be what you think
description: ''
tags: 'typescript, javascript'
cover_image: 'https://res.cloudinary.com/practicaldev/image/fetch/s--82YdvGeH--/c_imagga_scale,f_auto,fl_progressive,h_420,q_auto,w_1000/https://dev-to-uploads.s3.amazonaws.com/uploads/articles/63mpj0ktn4x8k7mrftkt.png'
published: false
id: 702771
---

I've done a lot of interviews on the interviewer-side recently and one thing surprised me:
Many developers get the concept of `as string` in TypeScript wrong; thinking they are save now that they use it - but really aren't. 😕
So what does `as string` or more generally `as MyType` actually do? And how do I fix the gap it leaves?

## tl;dr

`as` keyword only changes _compile_-time behavior. You essentially tell the compiler to stop complaining. Runtime behavior stays unchanged. If you need runtime safety, use techniques like polymorphism, custom type guards, or `instanceof` type guards, or assertion functions.

## What `as string` actually does

Or _Type Assertion_ vs _Type Casting_.

Many developers know Type Casting from languages like Java. They have a `double` and want to make it an `int`, so they do `int myInt = (int) myDouble;`. In Java this is fine and actually changes the behavior _at runtime_. But here's the thing: **TypeScript doesn't!**

Instead, TypeScript's `as MyType` only changes _compile time_ behavior! It's like saying to the compiler "Compiler, believe me please. This is a `MyType`. So stop complaining." Nothing has changed for the resulting JavaScript code after compiling. In other words, we're still completely vulnerable at _runtime_. That's why TypeScript actually uses the term _Type Assertion_ intead of _Type Casting_.

### Example Time

Let's look at an example where things go wrong. The code will result in a `addLemon is not a function` runtime error.

```typescript
interface Tea {
  addLemon(): void;
}
interface Coffee {
  addSugar(): void;
}

function addExtra(hotBeverage: Tea | Coffee) {
  (hotBeverage as Tea).addLemon();
}

const actuallyCoffee: Coffee = {
  addSugar: () => {},
};
addExtra(actuallyCoffee);
```

We define interfaces `Tea` and `Coffee` with different methods on them. Next, we incidentally grab ourselves a Coffee (what a grave mistake), and pass it to the `addExtra` function. This function however misuses `as Tea`, essentially assuming/asserting everything we pass to the function is always tea. The result is a runtime error saying `TypeError: hotBeverage.addLemon is not a function`. Making it into production might easily cost us a few thousand bucks to fix and a lot of customer frustration because they can't drink their morning coffee. At least your customers can still drink `Tea`. 😉

Admittedly, this example seems slightly artificial but there are enough examples in the real world where similar things happen.

## How to fix the gap

Depending on what you want from a business perspective, the solution might be using _Type Guards_. Another approach for the above is using polymorphism.

### Polymorphism

Wikipedia's intro sentence for [Polymorphism](<https://en.wikipedia.org/wiki/Polymorphism_(computer_science>) is

> In programming languages and type theory, polymorphism is the provision of a single interface to entities of different types [...]

That is, instead of having an `addExtra` function, we might change the code into the following.

```typescript
interface Tea {
  add(extra: string): void;
}
interface Coffee {
  add(extra: string): void;
}

const actuallyCoffee: Coffee = {
  add: (extra: string) => {},
};
actuallyCoffee.add("sugar");
```

This allows also adding different things into our tea and coffee. In case, we want to restrict what can be added into the coffee, we can use TypeScript's [Literal Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types).

```typescript
interface Coffee {
  add(extra: "sugar"): void;
}
// ...
actuallyCoffee.add("sugar"); // still works
actuallyCoffee.add("lemon"); // compile error: '"lemon"' is not assignable to parameter of type '"sugar"'
```

### Type Guards

Type Guards are essentially `as` but _with runtime check_! Note however that since we're writing the type guard ourselves, there is still some risk of getting it wrong. So be careful.

Fixing our above example we might the following

```typescript
// ...

// type guard
function isTea(x: Tea | Coffee): x is Tea {
  if ((x as Tea).addLemon) {
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

addExtra(actuallyCoffee); // adds sugar without any errors
```

The type guard `isTea` is a function that receives a single parameter, returns a boolean, and is marked with `x is Tea`. More generally, it's marked with `myParameter is MyType`. TypeScript will use it to narrow down the type in the `if-statement` of `addExtra`

Note that we use `x as Tea` here in a responsible manner - to allow us to access the property `addLemon` and check its existence! In practice, I recommend further checks to determine the type. For example, is `addLemon` really a function, and not a string, etc.

#### TypeScript trusts us in writing proper Type Guards

The fact that TypeScript trusts us in implementing a proper type guard can be shown easily.
Take a look at the following, obviously wrong type guard. TypeScript believes in us and won't budge - the executed JavaScript will.

```typescript
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

### `instanceof` Type Guard

This one is only usable if `Tea` and `Coffee` are classes. In our case, they are only interfaces. But let's pretend for a moment anyway. In this case, we don't need to write a manual type guard. Instead, `instanceof Tea` and `instanceof Coffee` is all we need to use TypeScript's powers.

```typescript
class Tea {
  addLemon() {}
}

function addExtra(hotBeverage: Tea | Coffee) {
  if (hotBeverage instanceof Tea) {
    hotBeverage.addLemon(); // TS narrows down the type to Tea
  } else {
    hotBeverage.addSugar(); // TS narrows down the type to Coffee
  }
}
```

## What about `<MyType>`?

You might have seen Type Assertions in the form `const tea = <Tea>{color: 'green'}` before. This is the same as `as Tea`. So again, it doesn't change any runtime behavior, leaving ourselves open for bad surprises.

The `<MyType>` syntax has been deprecated because it becomes confusing in the context of `.jsx` or `.tsx` files. Both use the same syntax. Thus, TypeScript recommends always using `as MyType`.

To clarify, take a look at this.

```typescript
const tea = <Tea>{ color: 'green' };
return <Tea>{tea.color}</Tea>
```

Here we have a component called `Tea` but also a type called `Tea` with a type assertion to the `Tea` type. If you are confused now, that's intended. 🙃 With `as Tea` it's more readable.

```typescript
const tea = {} as Tea;
return <Tea>{tea}</Tea>;
```

## Closing

Wanna learn more about TypeScript? Then hit subscribe on the blog, and join me on [Twitch fadeoutsama](https://www.twitch.tv/fadeoutsama/schedule) for our TypeScript Teatime! Looking forward to see you!

## Recommended Reading

- [basarat's TypeScript book - Type Assertions](https://basarat.gitbook.io/typescript/type-system/type-assertion)
- [TypeScript docs - Type Guards](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-guards-and-differentiating-types) (deprecated page but there's no official replacement about Type Guards at time of writing)
- [TypeScript docs - instanceof Type Guards](https://www.typescriptlang.org/docs/handbook/advanced-types.html#instanceof-type-guards)
