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


There are many libs that will do this for you. I aim to describe a specific one called [Joi](https://github.com/hapijs/joi).  Throughout this article we will take the following journey together:

- **Have a look** at Joi's features
- **See how** we can use Joi in the backend in a Request pipeline, yes we will try to build a middleware for Express in Node.js
- **Look at** ready made middleware, as fun as it is to build your own code, someone who has already built one is great, they might have thought of edge cases you haven't and you really don't have the time to build it, or do you?

## Introducing Joi
Installing Joi is quite easy. We just need to type:

```
npm install joi
```

After that we are ready to use it. Let's have a quick look at how we use it. First thing we do is import it and then we set up some rules, like so:

```js
const Joi = require('joi');

const schema = Joi.object().keys({
    name: Joi.string().alphanum().min(3).max(30).required(),
    birthyear: Joi.number().integer().min(1970).max(2013),
});

const dataToValidate = {
  name 'chris',
  birthyear: 1971
}

const result = Joi.validate(dataToValidate, schema);

result.error // result.error == null means it's valid
```
What we are looking at above is us doing the following:
- constructing a schema, our call to `Joi.object()`,
- validating our data, our vall to `Joi.validate()` with `dataToValidate` and `schema` as input parameters

Ok, now we understand the basic motions. What else can we do?

Well Joi supports all sorts of primitives as well as Regex and can be nested to any depth. Let's list some different constructs it supports:

- **string**, this says it needs to be of type string, and we use it like so `Joi.string()`
- **number**, Joi.number() and also supporting helper operations such as `min()` and `max()`, like so `Joi.number().min(1).max(10)`
- **required**, we can say wether a property is required with the help of the method `required`, like so `Joi.string().required()`
- **any**, this means it could be any type, usually we tend to use it with the helper `allow()` that specifies what it can contain, like so,  `Joi.any().allow('a')`
- **optional**, this is strictly speaking not a type but has an interesting effect. If you specify for example `prop : Joi.string().optional`. If we don't provide `prop` then everybody's happy. However if we do provide it and make it an integer the validation will fail

## Building a middleware with Joi

## Be the TV Chef

## Summary

