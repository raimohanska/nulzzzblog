I'll illustrate the problems I've encountered implementing lazy-loading features in React components using component state.

Here's a simple component that loads some data, based on an `id` passed in props:

```jsx
class NaiveClassFetcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      data: "loading"
    };
    fetchData(props.id).then(data => this.setState({ data }));
  }

  render() {
    return <span>{this.state.data}</span>;
  }
}
```

This works as long as the component will never re-render using a different `id` prop. Well if it does, we can simple use the
`componentWillUpdate` lifecycle method, like here.

```jsx
class SmarterClassFetcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      data: "loading"
    };
    fetchData(props.id).then(data => this.setState({ data }));
  }

  componentWillUpdate(newProps) {
    // Must check if differs, otherwise will fetch again after
    // setState calls too!
    if (newProps.id !== this.props.id) {
      fetchData(newProps.id).then(data => this.setState({ data }));
    }
  }

  render() {
    return <span>{this.state.data}</span>;
  }
}
```

Nice! Now it works. I absolutely must check if the new `id` is different from the old one, or the component will go into an
endless loading loop though. Also, I've essentially duplicated my fetching code. Let's refactor!

```jsx
class RefactoredClassFetcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      data: "loading"
    };
    this.fetchMyData();
  }

  fetchMyData() {
    fetchData(this.props.id).then(data => this.setState({ data }));
  }

  componentWillUpdate(newProps) {
    if (newProps.id !== this.props.id) {
      this.fetchMyData();
    }
  }

  render() {
    return <span>{this.state.data}</span>;
  }
}
```

Alas, it fails! The fetch is always one step behind. This is because when fetchMyData is called, the `id` counter is still
at its previous value. We can fix this of course, as in

```jsx
class FixedRefactoredClassFetcher extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      data: "loading"
    };
    this.fetchMyData(props.id);
  }

  fetchMyData(id) {
    // Must not use this.props here, because they can lie!
    fetchData(id).then(data => this.setState({ data }));
  }

  componentWillUpdate(newProps) {
    // Must check if differs, otherwise will fetch again after
    // setState calls too!
    if (newProps.id !== this.props.id) {
      this.fetchMyData(newProps.id);
    }
  }

  render() {
    return <span>{this.state.data}</span>;
  }
}
```

Now it works again, though I must be very careful in what I do in the `fetchData` method. If I make the mistake of using props 
(or calling another method using props) I'm in danger of fetching data based on outdated inputs.

Scary huh?

Here's the same thing implemented using React Hooks.

```jsx
const HooksFetcher = ({id}) => {
  // Use local state, with the initial value of "loading" that's used when the component is mounted
  const [data, setData] = useState("loading")
  // Fetch data asynchronously when component is rendered, but *only when the id prop changes*.
  // It's important to pass an array of props/variables to check, to prevent unnecessary calls to fetchData.
  useEffect(() => { fetchData(id).then(setData) }, [id])
  // Render value from local state
  return <span>{data}</span>;
}
```

You can also extract the lazy-loading thingie into a custom hook:

```jsx
const JuicyHooksFetcher = ({ id }) => {
  const data = useLazyLoad(() => fetchData(id), id)
  return <span>{data}</span>;
}

const useLazyLoad = (fetch, ...idProps) => {
  const [data, setData] = useState("loading")
  useEffect(() => { fetch().then(setData) }, idProps)
  return data
}
```

Notice that the `useLazyLoad` hook is universal and can be used across all of your components. This is a feat not easily performed using lifecycle methods. Yes, you can write a component wrapper for that but that doesn't compose very nicely (what if you have to fetch 3 things lazily?). 

In this approach, there are no classes, methods or `this`. The code can be analyzed statically to verify there are no typos. All references to names are "clickable", because they are in scope instead of behind a "this".

I'd argue that it's pretty hard to get the class-based solution to work correctly and that there's a multitude of ways to fail. Even when writing these examples today I got, once again, surprised about the `setState` call triggering yet another `componentWillUpdate` call, necessitating the equality check in the latter method.

To simplify things a bit, error handling is ignored in the example, as well as showing the "loading" text on subsequent fetches. To add those, you can modify the new `useLazyLoad` hook to look like this:

```jsx
const useLazyLoad = (fetch, ...idProps) => {
  const [data, setData] = useState("loading");
  useEffect(() => {
    setData("loading")
    fetch().then(setData).catch(() => setData("error"));
  }, idProps);
  return data;
};
```

If you want to fool around with some running code, its all [here](https://codesandbox.io/s/kqn1vn9jr)
