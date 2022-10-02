---
title: 3 Ways To Deep Clone / Copy Objects In JavaScript And TypeScript
description: TODO
tags: "typescript, javascript"
cover_image: ""
published: false
id: 709324
---

Cloning (aka copying) objects is a common task in programming. JS and TS provide multiple ways to achieve this. Let's take a look at 3 common ways to deep copy an object.

## tl;dr

- manual painful copying property by property
- spread operator `{ ...originalObj, nested: { ...originalObj.nested } }`
- lodash cloneDeep `_.cloneDeep(originalObj);`

## The Path of Pain - Manual Cloning

You probably came here because you want a better way than this. So feel free to jump to the next part to see the real action. 😃

```typescript
const tea = {
  color: "green",
  rating: 9001,
  nestedTea: {
    color: "black",
    rating: 9000,
  },
};

const teaClone = {
  color: tea.color,
  rating: tea.rating,
  nestedTea: {
    color: tea.nestedTea.color,
    rating: tea.nestedTea.rating,
  },
};

tea === teaClone; // false
tea.nestedTea === teaClone.nestedTea; // false
```

### Shallow vs Deep Copying

Note that we need to completely write out any nested objects in our original tea as well! Failing to do so means that `tea.nestedTea` and `teaClone.nestedTea` are the identical object (as reported by `===` comparison). This is the difference between _shallow copying_ (sometimes just called _copying_), and _deep copying_.

Basic datatypes like `string`, `boolean` and `number` but also `null` and `undefined` are not passed by object references. As such, they are "automatically deep copied". Any higher level datatype however is passed by reference. This includes objects of course, arrays, and also functions and arrow functions.

## Spread Operator

The [spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#spread_in_object_literals) copies all properties from one object to another in a shallow fashion. We can use it to relieve ourselves from the manual path of pain.

```typescript
const tea = {
  color: "green",
  rating: 9001,
  nestedTea: {
    color: "black",
    rating: 9000,
  },
};

const teaClone = {
  ...tea,
  nestedTea: {
    ...tea.nestedTea,
  },
};

tea === teaClone; // false
tea.nestedTea === teaClone.nestedTea; // false
```

That's a lot better. Particularly, it scales much better with the number of non-object properties.
Be careful though: Nested objects need to be spread themselves. Again, failing to do so would result in a shallow copy, i.e. `tea.nestedTea === teaClone.nestedTea` being true. That's not what we want here so we have to spread `nestedTea` as well.

## Lodash - The Swiss Army Knife

I can't even count the number of times I shouted out "I never knew Lodash could do that! That's so cool!" With weekly over 40 million downloads on npm, many developers seem to feel the same. The library [Lodash](https://lodash.com/) not only provides helpful functions for working with arrays or collections but also for objects. [cloneDeep](https://lodash.com/docs/4.17.15#cloneDeep) and it's shallow-copy equivalent [clone](https://lodash.com/docs/4.17.15#clone) make life easy.

```typescript
import { cloneDeep } from "lodash";

const tea = {
  color: "green",
  nestedTea: {
    color: "black",
  },
};

const teaClone = cloneDeep(tea);

tea === teaClone; // false
tea.nestedTea === teaClone.nestedTea; // false
```

A nice side-effect is better communication of intent compared to object spreading. Note though that it's not vanilla JavaScript, so we need to include it in our project setup.

### How to include Lodash

To use Lodash, we need to include it in our project. How to include lodash depends on the project. Basically the question: Is it a NodeJS project or a Browser-based project?
TODO

```bash
npm install lodash

# If you use TypeScript you also need
npm install --save-dev @types/lodash
```

In case, you don't want to include all of Lodash because of your [bundle size](https://bundlephobia.com/result?p=lodash@4.17.21), you can also include only `lodash.cloneDeep` like so

```bash
npm install lodash.deepClone

# If you use TypeScript you also need
npm install --save-dev @types/lodash.deepClone
```

A different approach is including Lodash from a CDN like [JSDelivr](https://www.jsdelivr.com/package/npm/lodash-es) or [cdnjs](https://cdnjs.com/libraries/lodash.js) by directly including it inside your HTML header.

```html
<!-- Better click the copy button on the CDN website! -->
<script
  src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.21/lodash.min.js"
  integrity="sha512-WFN04846sdKMIP5LKNphMaWzU7YpMyCU245etK3g/2ARYbPK9Ub18eG+ljU96qKRCWh+quCY7yefSmlkQw1ANQ=="
  crossorigin="anonymous"
  referrerpolicy="no-referrer"
></script>
```

After the library is included, you can use it as we've done before, i.e.

```typescript
import { cloneDeep } from "lodash";
const teaClone = cloneDeep(tea);
```

## Closing

Wanna learn more about TypeScript? Then hit subscribe on the blog, and join our [Twitch TypeScriptTeatime](https://www.twitch.tv/typescriptteatime/schedule) for Live Coding Fun! Looking forward to see you!
