```
                                          💥           ⭐️            💥      ✨
                             ✨                                ✨
  ███████╗████████╗ █████╗ ████████╗███████╗██████╗ ██╗   ██╗██╗   ✨
  ██╔════╝╚══██╔══╝██╔══██╗╚══██╔══╝██╔════╝██╔══██╗╚██╗ ██╔╝██║
  ███████╗   ██║   ███████║   ██║   █████╗  ██████╔╝ ╚████╔╝ ██║     ⭐️    ✨
  ╚════██║   ██║   ██╔══██║   ██║   ██╔══╝  ██╔══██╗  ╚██╔╝  ╚═╝
  ███████║   ██║   ██║  ██║   ██║   ███████╗██║  ██║   ██║   ██╗     ✨  💥
  ╚══════╝   ╚═╝   ╚═╝  ╚═╝   ╚═╝   ╚══════╝╚═╝  ╚═╝   ╚═╝   ╚═╝        ✨
                                         ✨                 ✨   ⭐️
  SURPRISE-FREE STATE MANAGEMENT!                💥
                                                              ✨
```

[![Version](https://img.shields.io/npm/v/statery?style=for-the-badge)](https://www.npmjs.com/package/statery)
[![CI](https://img.shields.io/github/workflow/status/hmans/statery/CI?style=for-the-badge)](https://github.com/hmans/statery/actions?query=workflow%3ACI)
[![Downloads](https://img.shields.io/npm/dt/statery.svg?style=for-the-badge)](https://www.npmjs.com/package/statery)
[![Bundle Size](https://img.shields.io/bundlephobia/min/statery?label=bundle%20size&style=for-the-badge)](https://bundlephobia.com/result?p=statery)

### Features 🎉

- Simple, **noise- and surprise-free API**. Check out the [demo]!
- **Extremely compact**, both in bundle size as well as API surface (2 exported functions!)
- Fully **tested**, fully **typed**!
- **Designed for React** (with functional components and hooks), but can also be used **without it**.

### Non-Features 🧤

- Doesn't use **React Context** (but you can easily use it to provide a context-specific store!)
- Provides a simple `set` function for updating a store and not much else (have you checked out the [demo] yet?) If you want to use **reducers** or libraries like [Immer], these can easily sit on top of your Statery store.
- Currently no support for (or concept of) **middlewares**, but this may change in the future.
- While the `useStore` hook makes use of **proxies**, the store contents themselves are never wrapped in proxy objects. (If you're looking for a fully proxy-based solution, I recommend [Valtio].)
- **React Class Components** are not supported (but PRs welcome!)

### Comparison to Zustand

[Zustand](https://github.com/pmndrs/zustand) is a lovely minimal state management library for React, and Statery may feel very similar to it. Here are the key differences:

- In Zustand, **data and the functions that modify it are all stored in the same object**. I believe this to be architecturally unwise (the README even warns the user not to accidentally overwrite their store's API.) Furthermore, this leads to Zustand [not being able to infer the type](https://github.com/pmndrs/zustand/blob/main/docs/guides/typescript.md) of the state data from the `create` function's first argument, forcing you to provide it explicitly (including the API, because it's part of the state... and so on.)

  Statery doesn't share these problems; **stores _only_ contain data**. The store's type is inferred from the initial state passed into `makeStore`. You can either modify the state directly through `set`, or – the recommended approach – export an API for your store as a set of functions (see the examples in this document or the [demo] for what this looks like.)

- When accessing data using Zustand's `useStore` hook, **it expects you to provide a selector function** that returns the data you want. Statery's `useStore` instead uses a **transparent proxy** that automatically registers all state properties you acceess. **You don't have to do anything special** to use this, and you never have to worry about causing re-renders for updated data your component isn't actually interested in.

- Not that it matters much with packages this tiny, but Statery is _even smaller_ than Zustand (at roughly 50% the size.)

## SUMMARY

```tsx
import { value makeStore, value useStore } from "statery"

const store = makeStore({ counter: 0 })

const increment = () =>
  store.set((state) => ({
    counter: state.counter + 1
  }))

const Counter = () => {
  const { counter } = useStore(store)

  return (
    <div>
      <p>Counter: {counter}</p>
      <button onClick={increment}>Increment</button>
    </div>
  )
}
```

## BASIC USAGE

### Adding it to your Project

```
npm add statery
yarn add statery
pnpm add statery
```

### Creating a Store

Statery stores wrap around plain old JavaScript objects that are free to contain any kind of data:

```ts
import { value makeStore } from "statery"

const store = makeStore({
  player: {
    id: 1,
    name: "John Doe",
    email: "john@doe.com"
  },
  gold: 100,
  wood: 0,
  houses: 0
})
```

### Updating the Store

Update the store contents using its `set` function:

```tsx
const collectWood = () =>
  store.set((state) => ({
    wood: state.wood + 1
  }))

const buildHouse = () =>
  store.set((state) => ({
    wood: state.wood - 10,
    houses: state.houses + 1
  }))

const Buttons = () => {
  return (
    <p>
      <button onClick={collectWood}>Collect Wood</button>
      <button onClick={buildHouse}>Build House</button>
    </p>
  )
}
```

Updates will be shallow-merged with the current state, meaning that top-level properties will be replaced, and properties you don't update will not be touched.

### Reading from a Store (with React)

Within a React component, use the `useStore` hook to read data from the store:

```tsx
import { value useStore } from "statery"

const Wood = () => {
  const { wood } = useStore(store)

  return <p>Wood: {wood}</p>
}
```

When any of the data your components access changes in the store, they will automatically re-render.

### Reading from a Store (without React)

A Statery store provides access to its current state through its `state` property:

```ts
const store = makeStore({ count: 0 })
console.log(store.state.count)
```

You can also imperatively [subscribe to updates](#subscribing-to-updates-imperatively).

## ADVANCED USAGE

### Deriving Values from a Store

Just like mutations, functions that derive values from the store's state can be written as standalone functions:

```tsx
const canBuyHouse = ({ wood, gold }) => wood >= 5 && gold >= 5
```

These will work from within plain imperative JavaScript code...

```tsx
if (canBuyHouse(store.state)) {
  console.log("Let's buy a house!")
}
```

...mutation code...

```tsx
const buyHouse = () =>
  store.set((state) =>
    canBuyHouse(state)
      ? {
          wood: state.wood - 5,
          gold: state.gold - 5,
          houses: state.houses + 1
        }
      : {}
  )
```

...as well as React components, which will automatically be rerendered if any of the underlying data changes:

```tsx
const BuyHouseButton = () => {
  const proxy = useStore(store)

  return (
    <button onClick={buyHouse} disabled={!canBuyHouse(proxy)}>
      Buy House (5g, 5w)
    </button>
  )
}
```

### Forcing a store update

When the store is updated, Statery will check which of the properties within the update object are actually different objects (or scalar values) from the previous state, and will only notify listeners to those properties.

In some cases, you may want to force a store update even though the property has not changed to a new object. For these situations, the `set` function allows you to pass a second argument; if this is set to `true`, Statery will ignore the equality check and notify all listeners to the properties included in the update.

Example:

```tsx
const store = makeStore({
  rotation: new THREE.Vector3()
})

export const randomizeRotation = () =>
  store.set(
    (state) => ({
      rotation: state.rotation.randomRotation()
    }),
    true
  )
```

### Subscribing to updates (imperatively)

Use a store's `subscribe` function to register a callback that will be executed every time the store is changed.
The callback will receive both an object containing of the changes, as well as the store's current state.

```ts
const store = makeStore({ count: 0 })

const listener = (changes, state) => {
  console.log("Applying changes:", changes)
}

store.subscribe(console.log)
store.set((state) => ({ count: state.count + 1 }))
store.unsubscribe(console.log)
```

### Async updates

Your Statery store doesn't know or care about asynchronicity -- simply call `set` whenever your data is ready:

```ts
const fetchPosts = async () => {
  const posts = await loadPosts()
  store.set({ posts })
}
```

### TypeScript support

Statery is written in TypeScript, and its stores are fully typed. `useStore` knows about the structure of your store, and if you're about to update a store with a property that it doesn't know about, TypeScript will warn you.

If the state structure `makeStore` infers from its initial state argument is not what you want, you can explicitly pass a store type to `makeStore`:

```tsx
const store = makeStore<{ name?: string }>({})
store.set({ name: "Statery" })
store.set({ foo: 123 })
```

## NOTES

### Stuff that probably needs work

- [ ] No support for middleware yet. Haven't decided on an API that is adequately simple.

### Prior Art & Credits

Statery was born after spending a lot of time with the excellent state management libraries provided by the [Poimandres](https://github.com/pmndrs) collective, [Zustand] and [Valtio]. Statery started out as an almost-clone of Zustand, but with the aim of providing an even simpler API. The `useStore` hook API was inspired by Valtio's very nice `useProxy`.

Statery is written and maintained by Hendrik Mans. [Get in touch on Twitter](https://twitter.com/hmans)!

[demo]: https://codesandbox.io/s/statery-clicker-game-hjxk3?file=/src/App.tsx
[zustand]: https://github.com/pmndrs/zustand
[valtio]: https://github.com/pmndrs/valtio
[immer]: https://github.com/immerjs/immer
