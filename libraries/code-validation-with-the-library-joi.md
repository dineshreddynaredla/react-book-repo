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
- **array**, we can check wether the property is an array of say strings, then it would look like this ` Joi.array().items(Joi.string().valid('a', 'b')` 
- **regex**, it supports pattern matching with RegEx as well like so `Joi.string().regex(/^[a-zA-Z0-9]{3,30}$/)`

The whole API for Joi is enormous. I suggest to have a look and see if there is a helper function that can solve whatever case you have that I'm not showing above [Joi API](https://github.com/hapijs/joi/blob/v14.3.1/API.md)

### Nested types
Ok so we have only showed how to declare a `schema` so far that is one level deep. We did so by calling the following:

```
Joi.object().keys({

});

```
This stated that our data is an object. Then we added some properties to our object like so:

```
Joi.object().keys({
  name: Joi.string().alphanum().min(3).max(30).required(),
  birthyear: Joi.number().integer().min(1970).max(2013)
});
```
Now, nested structures are really more of the same. Let's create an entirely new schema, a schema for a blog post, looking like this:
```js
const blogPostSchema = Joi.object().keys({
  title: Joi.string().alphanum().min(3).max(30).required(),
  description: Joi.string(),
  comments: Joi.array().items(
    Joi.object.keys({
      description: Joi.string(),
      author: Joi.string().required(),
      grade: Joi.number().min(1).max(5)
    })
  )
});
```
Note especially the `comments` property, that thing looks exactly like the outer call we first make and it is the same. Nesting is as easy as that.

## Building a middleware with Joi
Libraries like these are great but wouldn't it be even better if we could use them in a more seamless way, like in a Request pipeline? Let's have a look firstly how we would use `Joi` in an Express app in Node.js:

```js
const Joi = require('joi');

// set up Express is omitted

app.post('/blog', (req, res, next) => {
    
    // get the body
    const { body } = req;

    // define the validation schema
    const blogSchema = Joi.object().keys({
        title: Joi.string().required
        description: Joi.string().required(),
        authorId: Joi.number().required()
    });

    // validate the body
    const result = Joi.validate(body, blogShema);
    const { value, error } = result;
    const valid = error == null;
    if (!valid) {
      res.status(422).json({
        message: 'Invalid request',
        data: body
      })
    } else {
      const createdPost = await api.createPost(data);
    
      res.json({
        message: 'Resource created',
        data: createdPost
      })
    }
});
```
The above works. But we have to define the schema call `validate()` on each request to a specific route. It's, for lack of a better word, lacks elegance. We want something more slick looking. 

Let's see if we can't rebuild it a bit to a middleware. Middlewares in Express is simply something we can stick into the request pipeline whenever we need it. In our case we would want to try and verify our request and early on determine wether it is worth proceeding with it or abort it. 

So let's look at a middleware. It's just a function right:
```
const handler = (req, res, next) = { // handle our request }
const middleware = (req, res, next) => { // to be defined }

app.post(
  '/blog', 
  middleware,
  handler
)
```

## Be the TV Chef

## Summary

