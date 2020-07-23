---
id: serializabilityMiddleware
title: Serializability Middleware
sidebar_label: Serializability Middleware
hide_title: true
---

# Serializability Middleware

A custom middleware that detects if any non-serializable values have been included in state or dispatched actions, modeled after `redux-immutable-state-invariant`. Any detected non-serializable values will be logged to the console.

This middleware is added to the store by default by [`configureStore`](./configureStore.md) and [`getDefaultMiddleware`](./getDefaultMiddleware.md).

You can customize the behavior of this middleware by passing any of the supported options as the `serializableCheck` value for `getDefaultMiddleware`.

## Options

```ts
interface SerializableStateInvariantMiddlewareOptions {
  /**
   * The function to check if a value is considered serializable. This
   * function is applied recursively to every value contained in the
   * state. Defaults to `isPlain()`.
   */
  isSerializable?: (value: any) => boolean
  /**
   * The function that will be used to retrieve entries from each
   * value.  If unspecified, `Object.entries` will be used. Defaults
   * to `undefined`.
   */
  getEntries?: (value: any) => [string, any][]

  /**
   * An array of action types to ignore when checking for serializability.
   * Defaults to []
   */
  ignoredActions?: string[]

  /**
   * An array of dot-separated path strings to ignore when checking
   * for serializability, Defaults to ['meta.arg']
   */
  ignoredActionPaths?: string[]

  /**
   * An array of dot-separated path strings to ignore when checking
   * for serializability, Defaults to []
   */
  ignoredPaths?: string[]
  /**
   * Execution time warning threshold. If the middleware takes longer
   * than `warnAfter` ms, a warning will be displayed in the console.
   * Defaults to 32ms.
   */
  warnAfter?: number
}
```

## Exports

### `createSerializableStateInvariantMiddleware`

Creates an instance of the serializability check middleware, with the given options.

You will most likely not need to call this yourself, as `getDefaultMiddleware` already does so.

Example:

```js
import { Iterable } from 'immutable'
import {
  configureStore,
  createSerializableStateInvariantMiddleware,
  isPlain
} from '@reduxjs/toolkit'

// Augment middleware to consider Immutable.JS iterables serializable
const isSerializable = value => Iterable.isIterable(value) || isPlain(value)

const getEntries = value =>
  Iterable.isIterable(value) ? value.entries() : Object.entries(value)

const serializableMiddleware = createSerializableStateInvariantMiddleware({
  isSerializable,
  getEntries
})

const store = configureStore({
  reducer,
  middleware: [serializableMiddleware]
})
```

### `isPlain`

Checks whether the given value is considered a "plain value" or not.

Currently implemented as:

```ts
export function isPlain(val: any) {
  return (
    typeof val === 'undefined' ||
    val === null ||
    typeof val === 'string' ||
    typeof val === 'boolean' ||
    typeof val === 'number' ||
    Array.isArray(val) ||
    isPlainObject(val)
  )
}
```

This will accept all standard JS objects, arrays, and primitives, but return false for `Date`s, `Map`s, and other similar class instances.