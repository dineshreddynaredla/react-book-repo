# Hooks

Hooks is the latest pattern and experimental feature that's better than sliced bread. Everyone used to go next over [Render props](/patterns/render-props.md) but now it's all hooks. 

> Hooks are an upcoming feature that lets you use state and other React features without writing a class. They’re currently in React v16.8.0-alpha.0

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


## Selling point of Hooks
- **you can extract stateful logic from a component**, so it can be tested independently and reused. 
- **allows you to reuse stateful logic**, without changing your component hierarchy. This makes it easy to share Hooks among many components or with the community.

## Hooks - approach
Hooks let you `split one component into smaller functions` based on what `pieces are related` (such as setting up a subscription or fetching data), rather than forcing a split based on lifecycle methods

## Summary

### Further reading
- [Hooks documentation](https://reactjs.org/docs/hooks-overview.html)
- [Motivation behind hooks](https://reactjs.org/docs/hooks-intro.html#motivation)