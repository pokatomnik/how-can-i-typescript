# How to invoke constructor dynamically

Let's imagine we have some class

```typescript
class Foo {
  constructor(private a: string, private b: number) {}
}
```

Pretty simple, right?

Let's imagine we have a lot of classes with the same constructor signatures:

```typescript
class Bar {
  constructor(private a: string, private b: number) {}
}

class Baz {
  constructor(private a: string, private b: number) {}
}
```

And we need a function that makes an instance of any of them.

Typically, we just invoke a constructor Itself to make an instance of the particular class.

```typescript
const foo = new Foo('hello', 42);
```

But there are some cases when we can't.
So we need a function like this:

```typescript
const foo: Foo = make(Foo, 'hello', 42);
const bar: Bar = make(Bar, 'hello', 42);
const baz: Baz = make(Baz, 'hello', 42);
```

Typescript has a great feature that allows us to do that.

So let's describe _any_ constructor interface.

```typescript
interface IConstructor<T, A extends Array<unknown>> {
  new(args: ...A): T
}
```

Where `T` is the interface of the instance.
Don't worry about `A extends unknown`, your arguments will be of the type you specify.

Okay, now we have the interface. Now let's describe our function finally.

```typescript
function make<T, A extends Array<unknown>>(
  Class: IConstructor<T, A>,
  ...args: A
): T {
  return new Class(...args);
}
```

What if your constructor has no params. Not a problem:

```typescript
class NoParams {}
```

```typescript
const noParams = make(NoParams);
```

That's It! Check [full example](https://www.typescriptlang.org/play?#code/JYOwLgpgTgZghgYwgAgMIHsQGcxQK4JjpQA8AKgDTICCyEAHpCACZY1RRwCeJeIA1iHQB3EAD4xyAN4AoZMhARhACgB06uFADmWAFw0AlPrIBuGQF8ZMmH0LBMyALZx+EEnOSUPtBk1btOHj5BEXEZMWUEABs4AC9Y-QxsXAIiUkoaMSp1VU0dfWojT2kPKAgwPCgQBSVkaLjYtQ1tLAMzSxl6rDYAMXR0EvkETBx8QmJlAAcoYAA3OEhkOH1R0C0qabmFlAAjfRA8Rx3oA2lLeUm8HajgBGQtcuplU9l5N+QyiqrkMAALYCwuTM8g6FyuNzuDzAACFnoN3h9ypVqn8AaodsDkJYOl02NDNPDhskxmkpjN5otlshViB1shNhTdvtDscoC9zvTwbd7o84a8EZ9kT9-oC4JjQZzrtyodCAApRPBYADyij5Hnegu+qMBO2QAGpkABGcUWKxEnDIGD9ZAAXicLggyj66CoAHIrehXVRDW1OiMwMgdgS7c5XMp8VA3UGoF6jb7OjFusgAHLoWWaOCONhSHH+hRpjNZ-Sp9OcLO2+1hkuF1pmGQAenryHd-Vdfuw6CiEFUUXQWmUHtUUKeBnjjaN7awne7vf7g5lzzHTdd0bb5unPb7ymjQ95o7r44ATJON7Pt5pdzD5YqVY790A).
