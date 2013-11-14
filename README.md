# JavaScript Styleguide

We are in the business of maintaining code, therefore we should strive to write
code as consistently and maintainable as possible.

## Table of Contents

* [Tabs vs Spaces](#tabs-or-spaces)
* [Line Length](#line-length)
* [Whitespace](#whitespace)
* [Braces](#braces)
* [Parenthesis](#parenthesis)
* [Variable and Property Names](#variable-and-property-names)
* [Constructor Names](#constructor-names)
* [Named Functions](#named-functions)
* [Function Spacing](#function-spacing)
* [Ternary Spacing](#ternary-spacing)
* [Semi-colons](#semi-colons)
* [Variables](#variables)
* [Comma First](#commas)
* [Errors](#errors)
* [Return Early](#return-early)
* [`new` is Optional](#new-is-optional)
* [Inline Documentation](#inline-documentation)
* [Modifying Native Prototypes](#modifying-native-prototypes)
* [CoffeeScript](#coffeescript)

## Coding Guide

### Tabs or Spaces

Node.js core uses two space indents, so should we.

### Line Length

**80** characters max.

### Whitespace

No trailing whitespace allowed. Clean up after yourself.

### Braces

Place opening braces on the same line as the statement.

```js
if (areWeWritingJava)
{
  // hmmmm
}

if (nicelyDone) {
  // ok!
}
```

### Parenthesis

Leave space before and after the parenthesis for `if/while/for/catch/switch/do`

```js
// inconsistent
if(weAreHavingFun){
  console.log('Wheeee!');
}

// consistent
if (weAreHavingFun) {
  console.log('Wheeee!');
}
```

### Variable and Property Names

Use [lower camel case](http://en.wikipedia.org/wiki/camelCase#Variations_and_synonyms) and be descriptive. Aim for one word descriptions if possible.

```js
// inconsistent
var plan_query = model.findById(plan_id);

// consistent
var planQuery = model.findById(planId);
```

### Constructor Names

Constructors should use [upper camel case](http://en.wikipedia.org/wiki/camelCase#Variations_and_synonyms) to hint to its intended use.

```js
function Movie(id, title) {
  if (!(this instanceof Movie)) {
    return new Movie(id, title);
  }

  this.id = id;
  this.title = title;
};

var gravity = new Movie(1, 'Gravity');
```

### Named Functions

Name your functions. It aids in comprehension when coming back to it six months later and provides more descriptive stack traces.

```js
model.findOne(params, function(err, doc) {
  // not very helpful
});

model.findOne(params, function userCallback(err, doc) {
  // more helpful
});
```

### Function Spacing

When executing a function, leave no space between it's name and parens.

```js
eatIceCream (); // inconsistent

eatIceCream();  // consistent
```

Leave a space between your `function` declaration/expression parenthesis and the opening `{`.

```js
function eatIceCream(){
  // inconsistent
};

function eatIceCream() {
  // consistent
};
```

### Ternary Spacing

Break ternarys across multiple indented lines. It's easier to switch the values around when you come back to it later and realize a mistake.

```js
// less maintainable
var docs = Array.isArray(response) ? response : [response];

// more maintainable
var docs = Array.isArray(res)
  ? res
  : [res];
```

Also, no nested ternary expressions permitted. Business logic is often
confusing enough on its own. If necessary, break out the logic into additional
`if/else` blocks.

```js
// do not nest
var msg = first
  ? isValid(thing)
    ? 'ok'
    : 'fail'
  : 'confusing';

// better
var msg = 'not so confusing';

if (first) {
  msg = isValid(thing)
    ? 'ok'
    : 'fail';
}
```

### Semi-colons

Use em.

```js
function semis() {
  // codez
}

['property'].forEach(console.log) // TypeError: Cannot call method 'forEach' of undefined
```

```js
function semis() {
  // codez
};   // <--------

['property'].forEach(console.log) // property 0 [ 'property' ]
```

It's really great if you take the extra time to
[learn where they're _actually_ necessary](http://blog.izs.me/post/2353458699/an-open-letter-to-javascript-leaders-regarding),
you may just be surprised how few places there are. I personally think it's easier to
learn the rules and use them only where they're required, but for even mid-size teams, it's
simpler to just use 'em and move on.

### Variables

When declaring multiple variables, use "var first" style.

```js
var fix = true;
var nice = 'you best';
var woot = require('woot');
```

This makes for easier copy/paste later on and less futzing around with semi-colons and commas.

### Commas

Use [comma first](https://gist.github.com/isaacs/357981).

```js
// arrays
var array = [
    'my'
  , 'array'
  , 'is'
  , 'neat'
];

// nested arrays
var nestedArray = [
    ['this', 'is', 'a', 'nested', 'array']
  , ['this', 'is', 'too']
];

// objects with on property may be inlined
var simpleObject = { x: 1 };

// objects with more than one property should use comma first
var avgObject = {
    first: 1
  , second: 2
};

// nested objects
var nestedObjects = {
    my: 'object'
  , is: {
        nested: 'indeed'
      , you: 'see'
    }
};
```

## Errors

If you must throw an error, make sure what you throw is an instance of `Error`.

For example:

```js
if (!exists) {
  throw 'invalid!'; // not very helpful
}

if (!exists) {
  throw new Error('invalid!'); // more helpful
}
```

Each instance of `Error` has a `stack` property which aids in debugging.

```js
try {
  throw new Error('hmmmm');
} catch (err) {
  console.error(err.stack);
}
```

Can I throw:

- `strings`? nope
- `numbers`? no
- `arrays`? nadda
- `objects` that have a message property? forget it
- `up`? please not on me

[More context](http://www.devthought.com/2011/12/22/a-string-is-not-an-error/)

Same goes for callbacks. If an error condition is identified, pass an instance of `Error`
to the callback.

```js
app.use(function(req, res, next) {
  var err = failedValidation(req)
    ? new Error('could not parse request')
    : undefined;

  next(err);
});
```

### Return Early

Return early whereever you can to reduce nesting and complexity.

```js
function checkSomething(req, res, next) {
  if (req.params.id) {
    if (isValidId(req.params.id)) {
      find(req.params.id, function(err, doc) {
        if (err) return next(err);
        res.render('plan', doc);
      });
    } else {
      next(new Error('bad id'));
    }
  } else {
    next();
  }
};

// return early
function checkSomething(req, res, next) {
  if (!req.params.id) {
    return next();
  }

  if (!isValidId(req.params.id)) {
    return next(new Error('bad id'));
  }

  find(req.params.id, function(err, doc) {
    if (err) return next(err);
    res.render('plan', doc);
  });
};
```

### `new` is Optional

In constructors, first check if the function was called without the `new`
keyword and correct it if possible by returning a new instance of the
constructor and passing along the arguments.
This guards against programmer error with negligable downside.

```js
function Script(code) {
  this.code = code;
};

var script = Script('var x = 1');
console.log(script); // undefined

// better
function Script(code) {
  if (!(this instanceof Script)) {
    return new Script(code);
  }

  this.code = code;
};

var script = Script('var x = 1');
console.log(script); // { code: 'var x = 1' }
```

## Inline Documentation

Each function should have corresponding JSDoc-style comments that include
parameters and return values. Examples are always appreciated.

```js
/**
 * Adds two numbers.
 *
 * Example
 *
 *     var result = add(39, 12);
 *
 * @param {Number} num1
 * @param {Number} num2
 * @return {Number}
 */

function add(num1, num2) {
  return num1 + num2;
};
```

### Modifying Native Prototypes

Don't do it. It causes debugging headaches, breaks expected behavior and introduces potential incompatibility with new versions of V8.
There's always a better way.

```js
// please no
Array.prototype.contains = function(needle) {
  return this.indexOf(needle) > -1;
};

// For example, create a helper module with a function that accepts an array:

// helpers/array.js
exports.contains = function(array, needle) {
  return array.indexOf(needle) > -1;
};
```

## CoffeeScript

Is banned. Just learn JavaScript. Really. Its a simple language.

