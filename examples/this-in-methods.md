# `this` in methods

Let's imagine we have a simple class:

```typescript
class Observable<T> {
  private _value: T;

  private subscribers: Array<(value: T) => void> = [];

  constructor(value: T) {
    this._value = value;
  }

  public subscribe(subscriber: (value: T) => void) {
    this.subscribers = this.subscribers.concat(subscriber);
    return () => {
      this.subscribers.filter((currentSubscriber) => {
        return currentSubscriber !== subscriber;
      });
    };
  }

  public next(value: T) {
    this._value = value;
    this.subscribers.forEach((currentSubscriber) => {
      currentSubscriber(value);
    });
  }

  public getValue() {
    return this._value;
  }
}
```

We can set up some initial value in Its constructor, subscribe to value changes and push next value to It. It works totally fine:

```typescript
const obs = new Observable(0);
const unsubscribe = obs.subscribe((currentValue) => {
  console.log(`Hello, current value is ${currentValue}`);
});
obs.next(2);
```

Ok let's break It:

```typescript
const obs = new Observable(0);
const subscribe = obs.subscribe;
const next = obs.next;
const unsubscribe = subscribe((currentValue) => {
  console.log(`Hello, current value is ${currentValue}`);
});
next(2);
```

You probably know what is happening here: we lost a context and the methods we trying to invoke is incorrect.
Will It be checked by the Typescript? No, we didn't provide a very important thing: `this` type. In this use-case, `this` must point to a current `Observable` instance to make It work. It works okay until we save a method into a constant instead of calling It directly using class instance.

Okay, let's not `bind` this manually, but make types stronger:

```typescript
class Observable<T> {
  private _value: T;

  private subscribers: Array<(value: T) => void> = [];

  constructor(value: T) {
    this._value = value;
  }

  public subscribe(this: this, subscriber: (value: T) => void) {
    this.subscribers = this.subscribers.concat(subscriber);
    return () => {
      this.subscribers.filter((currentSubscriber) => {
        return currentSubscriber !== subscriber;
      });
    };
  }

  public next(this: this, value: T) {
    this._value = value;
    this.subscribers.forEach((currentSubscriber) => {
      currentSubscriber(value);
    });
  }

  public getValue(this: this) {
    return this._value;
  }
}
```

As you can see, the only change we made is `this: this` in the method signatures. It tells the compiler that these methods can be invoked **only** when `this` is pointing to `this`.
Wait for a second, but is `this` is a type and a class variable at the same time? Yes, that's correct.
Let's describe It more clearly.

```typescript
this: this
```

here we have left and right sides.
As usual, the left side is a variable name, and the right side describes Its type.

We can specify `this` type by describing Its type before method arguments and their types.
So this line tells the compiler that in these methods `this` must point directly to the current instance and right `this` points to a current instance type. For example, if a `Foo` class extended `Observable`, `this` is of type `Foo`, but not of type `Observable`.

Ok, let's check how does that tricky code work:

```typescript
const obs = new Observable(0);
const subscribe = obs.subscribe;
const next = obs.next;
// Error! "The 'this' context of type 'void' is not assignable to method's 'this' of type 'Observable<number>'."
const unsubscribe = subscribe((currentValue) => {
  console.log(`Hello, current value is ${currentValue}`);
});
// Error! "The 'this' context of type 'void' is not assignable to method's 'this' of type 'Observable<number>'."
next(2);
```

That's awesome, because `subscribe` and `next` have lost the context `obs` and the Typescript compiler will not allow these invocations.
