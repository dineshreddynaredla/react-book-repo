# Validating your data with the library Joi

Validation of data is an interesting topic we tend to write code that looks really horrible in the sense that it contains a lot of checks like this:

![](/assets/blur-chair-cheerful-160739.jpg)

```
if (!data.paramterX) {
  throw new Exception('parameterX missing')  
} 

try {
  let value = parseInt(data.parameterX);    
} catch (err) {
  throw new Exception('parameterX should be number')  
}

if(!/[a-z]/.test(data.parameterY)) {
  throw new Exception('parameterY should be lower caps text')  
}

```

I think you get the idea from the above cringeworty code. We tend to perform a lot of tests on our parameters to ensure they are the right `type` and/or their values contains the `allowed values`.

As developers we tend to feel really bad about code like this so we either start writing a lib for this or we turn to our old friend NPM and hope that some other developer have felt this pain and had too much time on their hands and made a lib that you could use. 


There are many libs that will do this for you. I aim to describe a specific one called `Joi`.  Throughout this article we will take the following journey together:

- **Have a look** at Joi's features
- **See how** we can use Joi in the backend in a Request pipeline, yes we will try to build a middleware for Express in Node.js
- **Look at** ready made middleware, as fun as it is to build your own code, someone who has already built one is great, they might have thought of edge cases you haven't and you really don't have the time to build it, or do you?

## Introducing Joi

## Building a middleware with Joi

## Be the TV Chef

## Summary

