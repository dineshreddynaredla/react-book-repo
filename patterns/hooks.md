# React Hooks - best thing since sliced bread?

> Hooks are an upcoming feature that lets you use state and other React features without writing a class component - functions FTW. 

Hooks is the latest pattern and experimental feature that's better than sliced bread ;). Everyone used to go nuts over [Render props](/patterns/render-props.md) but now it's all hooks. 

They’re currently in React v16.8.0-alpha.0 so you can try them out already :)

![](/assets/Screen Shot 2019-01-22 at 15.18.11.png)


It's an upcoming feature, it's an alpha and still we can't wait to use it, so why is that? 

## Problems Hooks are trying to address 
There are many problems Hooks are trying to address and solve. Here is a list covering some of them:

- **wrapper hell**, we all know the so called _wrapper hell_. Components are surrounded by layers of `providers`, `consumers`, `higher-order components`, `render props`, and other abstractions, exhausted yet? ;)

Like the whole wrapping itself wasn't bad we need to restructure our components which is tedious but most of all we loose track over how the data flows.

- **increasing complexity**, something that starts out small becomes large and complex over time, especially as we add lifecycle methods
- **life cycle methods does too many things**, 
 components might perform some data fetching in `componentDidMount` and `componentDidUpdate`. Same `componentDidMount` method might also contain some unrelated logic that sets up event listeners, with cleanup performed in `componentWillUnmount`

### Just create smaller components?
 
In many cases it’s not possible becuase: 
- **difficult to test**, stateful logic is all over the place, thus making it difficult to test
- **classes confuse both people and machines**, you have to understand how this works in JavaScript, you have to bind them to event handlers.
The distinction between function and class components in React and when to use each one _leads to disagreements_ even between experienced React developers.
- **minify issues**, classes present issues for today’s tools, too. For example, classes don’t minify very well, and they make hot reloading flaky and unreliable


### Selling point of Hooks
Hooks let you use more of React’s features **without classes**. Not only that we are able to create Hooks that will allow you to:

- **extract stateful logic from a component**, so it can be tested independently and reused. 
- **reuse stateful logic**, without changing your component hierarchy. This makes it easy to share Hooks among many components or with the community.

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

Ok, we name the first value in the array `counter` and the second value `setCounter`. The first value is the actual value that we can showcase in our `render` method. The second value `setCounter()` is a function that we can invoke and thereby change the value of `counter`. So in a sense, `setCounter(3)` is equivalent to writing:
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
```

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
Above we see that inside of our `useEffect` function we perform our side effect as usual but we can also set things up. We also see that we return a function. Said function will be invoked the last thing that happens. 

What we have here is _set up_ and _tear down_. So how can we use this to our advantage? Let's look at a bit contrived example first so we get the idea:

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
onMessage = (message) => {
 // do something with message
}

useEffect(() => {
  chatRoom.subscribe('roomId', onMessage)
  
  return () => {
    chatRoom.unsubscribe('roomId');
  }
})
```

## Can I create my own Hook?
Yes you can. With `useState` and `useEffect` the world is completely open. You can create whatever Hook you need. 

The mindset I want you to have is the following. Will my component have a state? Will I need to do a DOM manipulation or maybe an AJAX call? Most of all, is it something usable that more than one component can benefit from? If there are several yes here you can use a Hook to create it.

Let's look at some interesting candidates and see how we can use Hooks to build them out:

You could be creating things like:

 - **a modal**, this has a state that says wether it shows or not and will need to manipulate the DOM to add the modal itself and it will also need to clean up after itself when the modal closes
- **a feature flag**, feature flag will have a state where it says wether something should be shown or not, it will need to get its state initially from somewhere like `localStorage` and/or over HTTP
- **a cart**, a cart in an e-commerce app is something that most likely follows us everywhere in our app. We can sync a cart to `localStorage` as well as backend endpoint. 

### Feature flag
Let's try to sketch up our Hook and how it should be behaving:

```js
import { useState } from 'react';

function useFeatureFlag(flag) {
  const enabled = useState(Boolean(localStorage.getItem(flag)));

  const setFlag = (status) => {
    const flags = localStorage.getItem('flags') | {};
    
    localStorage.setItem('flags', { ...flags, flag: status }
   );
  }
  
 return [enabled, setFlag]; 
}
```

Now we have create our custom Hook, let's take it for a spin:
```
const MyComponent = () => {
  const [showExperiment1] = useFeatureFlag('experiment1');
  
  return (
    <div>Always show this</div>
    { showExperiment1 && 
    <div>Show this if feature flag is on</div>
    }
  )
}
```
Now, as you saw in our custom hook we are exposing two things, the state and a function that lets us change the state. However in `MyComponent` we are only taking advantage of the `state`. Well, that makes kind of sense right? I mean the aim was to only show a certain region in a component if it was true. 

The million dollar question is when would we use `setFlag` to change the value? Glad you asked ;) Imagine you have an admin page. On that admin page it would be neat if we could list all the flags and toggle them any way we want to. Let's write such a component:

```js
const AdminPage = () => {
  const flags = localStorage.getItem('flags') | {};
  
  const toggleFlag = (f) => {
    const [f, setFlag] = useFeatureFlag(flag);
    setFlag(!f); 
  }
  
  return (
    <React.Fragment>
    {Object.keys(flags).map(key => <div><button onClick={() => toggleFlag(flag)}>{flag}</button></div>)}
    </React.Fragment>
  )
}

```

## Summary
In this article we have tried to explain the background and the reason Hooks where created and what problems it was looking to address and hopefully fix.

We have learned the following, hopefully;):
- useState, is a Hook we can use to persist state in a functional component
- useEffect is also a Hook but for side effects
- we can use one or both of the mentioned Hook types and create really cool and reusable functionality, so go out there, be awesome and create your own hooks.

### Further reading
- [Hooks documentation](https://reactjs.org/docs/hooks-overview.html)
- [Motivation behind hooks](https://reactjs.org/docs/hooks-intro.html#motivation)