# Explicit Type Inference

Let's imaging we have some library and here's how do we import a function from It:

```typescript
import { someFunc } from 'some-library';
```

Based on Its typings, we know that `someFunc` returns a promise that wraps some type. Unfortunately, the type It returns isn't exported:

```typescript
// some-library code

interface ISomeType {
  title: string;
  name: string;
  isActive: string;
}

export function someFunc(): Promise<ISomeType> {
  const someData: ISomeType = {
    title: 'Foo',
    name: 'Bar',
    isActive: true,
  };
  return Promise.resolve(someData);
}
```

In our code we have a class:

```typescript
import { someFunc } from 'some-library';

export class SomeFuncData {
  private async initialize() {
    this.someData = await someFunc();
  }

  /*
    Oops, we have a problem, what type should be here?
    ISomeType isn't exported, how can we reach It?
  */
  private data: null | UNKNOWN_TYPE = null;

  // What does It return?
  public async getData(): UNKNOWN_TYPE {
    return this.data || (this.data = await this.initialize());
  }
}
```

Ok, what `UNKNOWN_TYPE` should be? How to unpack a promise type from that function?

## `infer` keyword to the rescue!

So we do know what promise type is, but we do not know what is a type argument of that promise.

Here's how can we unpack It, let's check the example and describe what all this stuff means.

```typescript
type PromiseType<T extends Promise<unknown>> = T extends Promise<infer R>
  ? R
  : never;
```

So, here we describe a new Type:

```typescript
type PromiseType
```

It's a generic type, It takes one type argument. This argument must be of Promise type (`unknown`, because It does not matter for now):

```typescript
type PromiseType<T extends Promise<unknown>>
```

After that, we must check if the value passed is of promise type and mark `Promise` type argument (`R`) as _inferrable_ using the `infer` keyword:

```typescript
type PromiseType<T extends Promise<unknown>> = T extends Promise<infer R>
```

Then, If `T` is of `Promise` type, we'll just return _inferrable_ type argument (`R`) or `never` otherwise:

```typescript
type PromiseType<T extends Promise<unknown>> = T extends Promise<infer R>
  ? R
  : never;
```

Why `never`? Because `never` in the Typescript is an empty type. It does not contain any types in It. And if the `PromiseType` type argument can't be treated as `Promise`, the Typescript checker can't infer It and we can't do anything with that type.

OK, let's check our class and replace `UNKNOWN_TYPE` with a correct type:

```typescript
import { someFunc } from 'some-library';

type PromiseType<T extends Promise<unknown>> = T extends Promise<infer R>
  ? R
  : never;

type SomeFuncReturnType = ReturnType<typeof someType>;

type SomeFuncDataType = PromiseType<SomeFuncReturnType>;

export class SomeFuncData {
  private async initialize() {
    this.someData = await someFunc();
  }

  private data: null | SomeFuncDataType = null;

  public async getData(): SomeFuncDataType {
    return this.data || (this.data = await this.initialize());
  }
}
```

OK, It seems, now we've done with that and now the `SomeFuncData` `data` field is of the correct type.

But what about `ReturnType`? What's that? Let me explain.

`infer` keyword can be used for unpacking any type arguments from any generic types. In this case, `ReturnType` just infers the function return type from a function.

Here is a short example:

```typescript
type FunctionReturnType<T extends Function> = T extends (
  ...args: any
) => infer R
  ? R
  : never;
```

We may not describe our own `ReturnType` type because It's already described in the inner Typescript `d.ts` file: `typescript/lib/lib.es5.d.ts`
