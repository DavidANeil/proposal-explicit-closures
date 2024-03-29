# proposal-explicit-closures

Blah blah blah
I want this syntax

```javascript
function myFunction(myParam)[console] {
    console.log(myParam);
}
```

Look what happens now:

```javascript
function myFunction(myParam)[] {
    console.log(myParam); // ReferenceError: console is not defined
}
```

### Why?

Because Chrome (and probably every browser) does a lot of work to minimize the closure it needs to store down to what is actually needed, this will make it not have to do that.

They already can't do any of that when `eval` or `new Function` are encountered, and so potentially a lot of data has to be held in memory.

Also those functions are considered insecure because they can arbitrarily access and modify the rest of the program.

Check this out:

```javascript
function saferEval(evalString)[eval] {
    return eval(evalString);
}
```

It can't access anything outside of its scope, so it is a lot safer. This includes access to `globalThis` and similar things, so it can't read cookies or make network requests either.

Also, we could do renaming

```javascript
function assertNotCircular(lists) {
    lists.forEach(function(lists)[lists: outerLists, ...] {
        if (lists === outerLists) {
            throw new Error('Circular entry found');
        }
    })
}
```

`[lists: outerLists]` binds the `lists` where the function is defined to the identifier `outerLists` inside the body of the function.
This is similar to object destructuring syntax.

ALTERNATE: Instead of object destructuring semantics, if we used object creation semantics, it would be `[outerLists: lists]`. We could then even do arbitrary calculations like `[outerLists: lists.filter(a => !!a)]`.

`[...]`, is the default, usually doesn't have to specified. In this case we could explicitly bind `[Error]`, but we don't need to.

### Arrow function syntax

Arrow functions will not support this syntax.

### Class syntax

```javascript
class Foo extends Bar {
    myMethod(myParam)[] {
    }

    constructor(myParam)[super] {
        super(myParam);
    }
}
```

### Expando Functions

Additional properties may not be added to an explicit closure function after declaration. This is to allow future extensions where such a function definition is shareable to other threads.

```javascript
function myFunction(a, b)[] {
    return a + b;
}
myFunction.docs = "abc"; // TypeError: Cannot assign to read only property 'docs' of object 'myFunction'
```

### Prototypes

Including an object also includes everything on its prototypes.

### `this`, `arguments`, etc.

`this`, and `arguments` (and any other special identifiers I am forgetting) are considered call-site parameters, so can be accessed even if they are not included in the explicit closure list.

An arrow function must list `[this]` if it wishes to include `this` from its parent in its closure. Indeed this could be considered the default for arrow functions.
Interestingly, a standard function could mimic this particular behavior of arrow functions by explicitly binding `[this]`.

### Pure functions

A function that does not bind any closure is guaranteed to be pure. Everyone loves this.

### Functions that are intended to be executed elsewhere

For example, using the Puppeteer library we have the following

```javascript
const transformData = (data) => {
  return data.map((point) => point * 2);
};
const result = await page.evaluate(() => {
  return window.myAppClient.doSomething(
    transformData(window.myAppClient.getData())
  );
});
```

This will break at runtime because the function being passed in is going to be stringified by Puppeteer and evaluated in a scope without access to `transformData`.

While

```javascript
page.evaluate(function()[] {
    ...
}
```

would be analyzable by static tools (such as TypeScript) to find the issue ahead of time.

### Implicits

This is not a proposal for implicit parameters. These values are bound at the time the function is declared, not at the callsite.

### Standard Library

No closure means no closure. No access to `JSON.parse`, no access to `Math.random`, no access to `NaN` or `Symbol` or `Object`.

This will be unfortunate when trying to do something like

```javascript
function factory(x)[] {
    return {a: x}
}
```

Because creating that object would require access to the `Object` class and its prototype.

Similarly, would this work? Would the engine be able to implicitly convert the `string` to a boxed `String` object in order to access the `.length` property?

```javascript
function factory(x)[] {
    return `(${x})`.length;
}
```

Saying "no closure means no closure" might be fine but the alternative of always including the "Primitives" (`Object`, `Number`, `Boolean`, `Array`, `String`, `null`, `undefined`, `BigInt`) is still being investigated.
`Map`, `Set`, `Symbol`, and friends would not need to be included, as there is no native syntax to produce those.

### Mutable values

Will this output `0` or `1`?

```javascript
let x = 0;
function f()[x: outerX] {
    return outerX;
}
x++;
console.log(f());
```

I _want_ the answer to be 1, but I imagine an engine implementer would hate me for wanting that.

## Alternatives

### Why not just use Function.prototype.bind?

1. This can be changed at the call-site.
2. While this can be used for the inclusionary purposes of this proposal, it doesn't work to exclude things. That is, `.bind` would be insufficient without a way of declaring a function that does not close over any variables.

### Create a syntax for declaring a function that does not close over any variables.

This seems just less flexible and less desirable.

## Prior Art

As far as I know, no other language lets you do this.
