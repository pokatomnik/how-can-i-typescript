# How to connect my component to some Context?

Let's imagine we have React class component.

```tsx
interface IMyComponentProps {
  title: string;
  description: string;
}

class MyComponent extends React.Component<IMyComponentProps> {
  public render() {
    return (
      <div>
        <h3>{this.props.title}</h3>
        <p>{this.props.description}</p>
      </div>
    );
  }
}
```

And we have a context:

```typescript
interface IContextValue {
  footer: string;
}

const MyContext = React.createContext<IContextValue>({
  footer: "Footer",
});
```

We need connect our component called `MyComponent` to `MyContext`.

There are three approaches:

## Using functional components

First, lets rewrite our component `MyComponent` this way:

```tsx
function MyComponent({ title, description }: IMyComponentProps) {
  return (
    <div>
      <h3>{title}</h3>
      <p>{description}</p>
    </div>
  );
}
```

Not a big deal for simple components (I assume It is).

Next step is the most easiest one, just do the following:

```tsx
function MyComponent({ title, description }: IMyComponentProps) {
  const { footer } = React.useContext(MyContext);
  return (
    <div>
      <h3>{title}</h3>
      <p>{description}</p>
      <p>{footer}</p>
    </div>
  );
}
```

## Using `context!` field

```tsx
class MyComponent extends React.Component<IMyComponentProps> {
  public static contextType = MyContext;

  public context!: React.ContextType<typeof MyContext>;

  public render() {
    return (
      <div>
        <h3>{this.props.title}</h3>
        <p>{this.props.description}</p>
        <p>{this.context.footer}</p>
      </div>
    );
  }
}
```

> Important note: you can connect your component to only one context this way.

## Using HOC (Higher-Order Component)

At first, your component should be able to accept context value as props:

```tsx
class MyComponent extends React.Component<IMyComponentProps & IContextValue> {
  public render() {
    return (
      <div>
        <h3>{this.props.title}</h3>
        <p>{this.props.description}</p>
        <p>{this.props.footer}</p>
      </div>
    );
  }
}
```

### Using hook:

```tsx
function connectToMyContext<O>(
  Component: React.ComponentType<O & IContextValue>
): (props: O) => React.ReactNode {
  return function ConnectedToMyContext(props: O) {
    const contextValue = React.useContext(MyContext);
    return <Component {...props} {...contextValue} />;
  };
}
```

### Using Consumer:

```tsx
function connectToMyContext<O>(
  Component: React.ComponentType<O & IContextValue>
): (props: O) => React.ReactNode {
  return function ConectedToMyContext(props: O) {
    return (
      <MyContext.Consumer>
        {(contextValue) => <Component {...props} {...contextValue} />}
      </MyContext.Consumer>
    );
  };
}
```

And connect your component:

```typescript
const MyComponentConnected = connectToMyContext<IMyComponentProps>(MyComponent);
```

> Important note: type `O` can't be inferred, you must specify It explicitly.

Here is the exampe how to use connected component:

```tsx
<MyComponentConnected title="foo" description="bar" />
<MyComponent title="foo" description="bar" />
```
