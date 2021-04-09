# How to make a wrapper of some function and preserve Its signature

Let's imagine we have a simple function.

```typescript
function plus(a: number, b: number) {
  return a + b;
}
```

We need to wrap this function into another one (for logging purposes maybe? No matter).

So the wrapper should have the same signature:

```typescript
function plusOneAndLog(a: number, b: number);
```

This function should be created this way:

```typescript
const plusOneAndLog = wrap(plus);
```

Looks simple, right? Let's make It.

```typescript
function wrap<R extends unknown, A extends Array<unknown>>(
  fn: (...args: A) => R
): (...args: A) => R {
  return function (...args: A): R {
    console.log("Here is the list of arguments:");
    console.log(...args);
    const result = fn(...args);
    console.log("Here is the result");
    console.log(result);
    return result;
  };
}
```

Now let's try:

```typescript
function plus(a: number, b: number) {
  return a + b;
}

console.log(plus(1, 2)); // 3

const plusOneAndLog = wrap(plus);

const result = plusOneAndLog(1, 2);

console.log(result; // 3
```
