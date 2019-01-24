# React Hooks - best thing since sliced bread?

> Hooks are an upcoming feature that lets you use state and other React features without writing a class component - functions FTW. 

Hooks is the latest pattern and experimental feature that's better than sliced bread ;). Everyone used to go nuts over [Render props](/patterns/render-props.md) but now it's all hooks. 

They’re currently in React v16.8.0-alpha.0

![](/assets/Screen Shot 2019-01-22 at 15.18.11.png)


It's an upcoming feature, it's an alpha and still we can't wait to use it, so why is that? 

## Problems Hooks are trying to address 
- **attach” reusable behavior to a component**, (for example, connecting it to a store)
[render props](/patterns/render-props.md) and TODO link **higher order** components try to solve these.
Which leads to:
_wrapper hell_ of components surrounded by layers of `providers`, `consumers`, `higher-order components`, `render props`, and other abstractions
These patterns require you to restructure your components when you use them, which can be cumbersome and make code harder to follow.
- **complex components become hard to understand**, something that starts out small becomes large and complex over time, especially as we add lifecycle methods
- **life cycle methods does too many things**, 
 - components might perform some data fetching in `componentDidMount` and `componentDidUpdate`. 
 - same `componentDidMount` method might also contain some unrelated logic that sets up event listeners, with cleanup performed in `componentWillUnmount`

> Just create smaller components?

In many cases it’s not possible to break these components into smaller ones because the stateful logic is all over the place. It’s also difficult to test them
- classes confuse both people and machines, you have to understand how this works in JavaScript, you have to bind them to event handlers.
The distinction between function and class components in React and when to use each one leads to disagreements even between experienced React developers.
However, we found that class components can encourage unintentional patterns that make these optimizations fall back to a slower path. Classes present issues for today’s tools, too. For example, classes don’t minify very well, and they make hot reloading flaky and unreliable


### Selling point of Hooks
- Hooks let you use more of React’s features without classes
- **you can extract stateful logic from a component**, so it can be tested independently and reused. 
- **allows you to reuse stateful logic**, without changing your component hierarchy. This makes it easy to share Hooks among many components or with the community.

## Hooks - approach
Hooks let you `split one component into smaller functions` based on what `pieces are related` (such as setting up a subscription or fetching data), rather than forcing a split based on lifecycle methods

## What is a hook?
Let's try to answer that question by working ourselves through some of the Hooks offered to us. 

### First example - useState
This Hook let's us use state inside of a function component. Yep I got your attention now right? Usually that's not possible and we need to use a class for that. Not anymore. Let's show what using `useState` Hook looks like:

```js
import { useState } from React;

const Counter = () => {
 const [counter, setCounter] = useState(0);
 
 return (
  <div>
    <button onClick={setCounter(count + 1)} >Increment</button>
  </div>
 )
}
```
Ok we see that we use the Hook `useState` by invoking it and we invoke it like so:

> useState(0)

This means we give it an initial value of 0. What happens next is that we get an array back that we do a destructuring on. Let's examine that closer:
> const [counter, setCounter] = useState(0);

Ok, we name the first value in the array `counter` and the second value `setCounter`. The first value is the actual value that we can showcase and the second value `setCounter()` is a function that we can invoke and thereby change the value of `counter`. So in a sense, `setCounter(3)` is equivalent to writing:
> this.setState({ counter: 3 })

Just to ensure we understand how to use it fully let's create a few more states:

```js
import { useState } from React;

const ProductList = () => {
 const [products, setProducts] = useState([{ id: 1, name: 'Fortnite' }]);
 const [cart, setCart] = useState([]);

 
 const addToCart = (p) => {
   const newCartItem = { ...p};
   setCart([...cart, newCartItem]);
 }
 
 return (
  <div>
    <h2>Cart items</h2>
    {cart.map(item => <div>{item.name}</div>)}
    <h2>Products</h2>
    {products.map(p => <div onClick={() => addToCart(p)}>{p.name}</div>)}
  </div>
 )
}
``` 

## Handling side-effects with a Hook
The Effect hook is meant to be used to perform side effects like for example HTTP calls.
It performs the same task as life cycle methods `componentDidMount`, `componentDidUpdate`, and `componentWillUnmount`.

Here is how we can use it:
```js
const ProductList = () => {
  const [products, setProducts] = useState([{ id: 1, name: 'Fortnite' }]);
 const [cart, setCart] = useState([]);

 useEffect(async() => {
   const products = await api.getProducts();
   setProducts(products);
 })
}
```

### Life cycle
Hooks replaces the needs for many life cycle methods in general so it's important for us to understand which ones. 

Let's discuss Effect Hooks in particular and their life cycle though.

The following is known about it's life cycle:

- By default, React runs the effects after every render
- Our effect is being run after React has flushed changes to the DOM — including the first render

### Accessing the DOM tree
Let's talk about when we access the DOM tree, to perform a side effect. If we are not using Hooks we would be doing so in the methods `componentDidMount` and `componentDidUpdate`. The reason is we cant use the `render` method cause then it would happen to early.

Let's show how we would use life cycle methods to update the DOM:
```js
componentDidMount() {
  document.title = 'Component started';
}

componentDidUpdate() {
  document.title = 'Component updated'
}
```
We see that we can do so using two different life cycle methods.

Accessing the DOM tree with an Effects Hook would look like the following:
```js
const TitleHook = () => {
  const [title, setTitle] = useState('no title');
  
  useEffect(() => {
   document.title = `App name ${title} times`;
  })
}

As you can see above we have access to `props` as well as `state` and the DOM. 

Let's remind ourselves what we know about our Effect Hook namely this:
> Our effect is being run after React has flushed changes to the DOM — including the first render

That means that two life cycle methods can be replaced by one effect.

### Handling set up/ tear down
Let's now look at another aspect of the `useEffect` hook namely that we can use it to clean up after ourselves. The idea for that is the following:
```
useEffect(() => {
 // set up 
 // perform side effect
 return () => {
   // perform clean up here
 }
});
```
Above we see that inside of our `useEffect` function we perform our side effect as usual but we can also set things up. We also see that we return a function, this function will be invoked the last thing that happens. 

What we have here is set up and tear down. So how can we use this to our advantage? Let's look at a bit contrived example first so we get the idea:

```
useEffect(() => {
  const id = setInterval(() => console.log('logging'));
  
  return () => {
    clearInterval(id);
  }
})
```
The above demonstrates the whole set up and tear down scenario but as I said it is a bit contrived. You are more likely to do something else like setting up a socket connect for example, e.g some kind of subscription, like so:

```
useEffect(() => {
  chatRoom.subscribe('roomId', onMessage)
  
  return () => {
    chatRoom.unsubscribe('roomId');
  }
})
```

## Best practices DOs and DON'Ts
TODO

## Can I create my own Hook?

## Let's code some Hooks

Let's first create a a project using CRA, like so:
> npx create-react-app react-hooks-demo

Ensure it's using React at least 16.7.0




## Summary

### Further reading
- [Hooks documentation](https://reactjs.org/docs/hooks-overview.html)
- [Motivation behind hooks](https://reactjs.org/docs/hooks-intro.html#motivation)