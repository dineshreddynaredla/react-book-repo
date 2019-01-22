# React Hooks - best thing since sliced bread?

> Hooks is the latest pattern and experimental feature that's better than sliced bread ;). Everyone used to go nuts over [Render props](/patterns/render-props.md) but now it's all hooks. 
Hooks are an upcoming feature that lets you use state and other React features without writing a class. They’re currently in React v16.8.0-alpha.0



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

## Are there more than one Hook?

## Can I create my own Hook?

## Let's code some Hooks

Let's first create a a project using CRA, like so:
> npx create-react-app react-hooks-demo

Ensure it's using React at least 16.7.0




## Summary

### Further reading
- [Hooks documentation](https://reactjs.org/docs/hooks-overview.html)
- [Motivation behind hooks](https://reactjs.org/docs/hooks-intro.html#motivation)