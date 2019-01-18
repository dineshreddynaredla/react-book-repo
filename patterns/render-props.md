# Using Render props in React - lifecycle begone!

What is Render props? Render props is way for us to create a component that provides some kind of data to another component. Why would we want that? Well imagine that we wanted to do something of the following:

- **fetching data**, wouldn't it be nice to have a component that abstracts away all of the mess of HTTP and just serves you the data when it's done?
- **paging**, imagine you pass in a data source, the page you want to see and the number of items on the page and out comes just the data you want
- **A/B testing**, as you launch an app into production you will eventually want to improve but you might not know the best way forward or you might want to release often and push the code to production but some feature is not yet ready to see the light of day, so you wan't to be able to conditionally decide wether something is visible or not

![](/assets/blaze-bonfire-burn-1374891.jpg)

If you have any of the scenarios above you have reusable functionality. With reusable functionality you most likely want to abstract that away into a function or a component, we're gonna opt for the latter. 

Wouldn't it be nice if we can create components for this functionality and just serves it up to some component. That child component would be unaware that it is being served up data. 

In a sense this resembles what we do with Providers but also how container components wraps presentation components. This all sounds a bit vague so let's show some markup how this could look:

```js
const ProductDetail = ({data}) => (
  <React.Fragment>
    <h2>{data.title}</h2>
    <div>{data.description}</div>
  </React.Fragment>
)

<Fetch url="some url where my data is" render={(data) => 
  <ProductDetail product={data.product} />
}>
```

As we can see above we have two different components `ProductDetail` and `Fetch`. `ProductDetail` just looks like a presentation component. `Fetch` on the other hand looks a bit different. It has a property `url` on it and it seems like it has a `render` property that ends up rendering our `ProductDetail`. 


## Render props explained
We can reverse engineer this and figure out how this works.

Let's have a look at the code again:

```
<Fetch url="some url where my data is" render={(data) => 
  <ProductDetail product={data.product} />
}>
```
Our `Fetch` component has an attribute `render` that seems to take a function that ends up producing JSX. Here is the thing the whole render-props pattern is about us invoking a function in our return method. Let me explain that by showing some code:

```
class Fetch extends React.Component {
  render() {
    return this.props.render();
  }  
}
```
This is what the pattern is, at its simplest. Of course the way we use the `Fetch` component means we at least need to send something into the `this.props.render()` call. Let's just extract the function invocation bit above and look at it:

```
(data) => <ProductDetail product={data.product} />
```
We can see above that that we need a parameter `data` and `data` seems to be an object. Ok, so where does `data` come from? Well thats the thing with our `Fetch` component, it does some heavy lifting for us namely carrying out HTTP calls. 

## Creating a component for HTTP
Let's add some life cycle methods to `Fetch` so it looks like this:

```js
// first draft 

class Fetch extends React.Component {
  state = {
    data: void 0,
    error: void 0
  }

  componentDidMount() {
    this.fetchData();
  }
  
  async fetchData() {
    try {
      const response = await fetch(this.props.url);
      const json = await response.json();
      this.setState({ data: json });
    catch (err) {
      this.setState({ error: err })
    }
  }

  render() {
    if (!this.state.data) return null;
    else return this.props.render(this.state.data);
  }  
}
```
Ok, now we have fleshed out our component a little. We've added the method `fetchData()` that makes HTTP calls given `this.props.url` and we can see that our `render()` method renders `null` if `this.state.data` isn't set, but if the HTTP call finished we invoke `this.props.render(data)` with our JSON response.

However it lacks three things:

- **handling error**, we should add logic to handle error
- **handling loading**, right now we render nothing if `fetch()` call hasn't finished, that isn't very nice
- **handling this.props.url**, this prop might not be set initially and it might be changed over time, so we should handle that

### Handling errors
We can easily handle this one by changing our render() method a little to cater to if `this.state.error` is set, after all we have already written logic that sets `this.state.error` in our catch clause in the `fetchData()` method.

Here goes:

```js
// second draft

class Fetch extends React.Component {
  state = {
    data: void 0,
    error: void 0
  }

  componentDidMount() {
    this.fetchData();
  }
  
  async fetchData() {
    try {
      const response = await fetch(this.props.url);
      const json = await response.json();
      this.setState({ data: json });
    catch (err) {
      this.setState({ error: err })
    }
  }

  render() {
    if(this.state.error) return this.props.error(this.state.error);
    if (this.props.render(this.state.data)) return null;
    else return null;
  }  
}
```
Above we added handling of `this.state.error` by invoking `this.props.error()`, so that is a thing we need to reflect once we try to use the `Fetch` component.

### handling loading
for this one we just need to add a new state `loading` and updated the `render()` method to look at said property, like so:

```js
// third draft

class Fetch extends React.Component {
  state = {
    data: void 0,
    error: void 0
  }

  componentDidMount() {
    this.fetchData();
  }
  
  async fetchData() {
    try {
      this.setState({ loading: true });
      const response = await fetch(this.props.url);
      const json = await response.json();
      this.setState({ data: json });
      this.setState({ loading: false });
    catch (err) {
      this.setState({ error: err })
    }
  }

  render() {
    const { error, data, loading } = this.state;
  
    if(loading) return <div>Loading...</div>
    if(this.state.error) return this.props.error(error);
    if (this.props.render(data)) return this.props.render(data);
    else return null;
  }  
}
```
Now, above we are a bit sloppy handling the loading, yes we add an `if` for it but what we render can most likely be improved using a nice component that looks like a spinner or a ghost image, so that's worth thinking about.

### Handling changes to this.props.url
It's entirely possible that this url can change and we need to cater to it, unless we plan on using the component like so `<Fetch url="static-url">`, in which case you should skip this section and look at the next section instead ;)

The React API recently changed, before the change we would've needed to add the life cycle method `componentWillReceiveProps` to look at if a prop changed, it's considered unsafe however so we must instead use 

```
componentDidUpdate(prevProps) {
  if (this.props.url && this.props.url !== prevProps.url) {
    this.fetchData(this.props.url);
  }
}
```
That's it, that's what we need, let's show the full code for this component:

```js
// final draft

class Fetch extends React.Component {
  state = {
    data: void 0,
    error: void 0
  }

  componentDidMount() {
    this.fetchData();
  }
  
  componentDidUpdate(prevProps) {
    if (this.props.url && this.props.url !== prevProps.url) {
      this.fetchData(this.props.url);
    }
  }

  async fetchData() {
    try {
      this.setState({ loading: true });
      const response = await fetch(this.props.url);
      const json = await response.json();
      this.setState({ data: json });
      this.setState({ loading: false });
    catch (err) {
      this.setState({ error: err })
    }
  }

  render() {
    if(this.state.loading) return <div>Loading...</div>
    if(this.state.error) return this.props.error(this.state.error);
    if (this.props.render(this.state.data)) return null;
    else return null;
  }  
}


```

 
## A/B Testing
## Creating a component for Paging

## Summary
### Further reading 

