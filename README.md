# proposal-explicit-closures

Blah blah blah
I want this syntax
```javascript
export function myFunction(myParam)[console] {
    console.log(myParam);
}
```

Look what happens now:
```javascript
export function myFunction(myParam)[] {
    console.log(myParam); // ReferenceError: console is not defined
}
```

### Why?

Because Chrome (and probably every browser) does a lot of work to minimize the closure it needs to store to what is actually needed, this will make it not have to do that.

They can't do any of that when `eval` or `new Function` are encountered, and so potentially a lot of data has to be held in memory.

Also those functions are considered insecure because they can arbitrarily access and modify the rest of the program.

Check this out:
```javascript
export function saferEval(evalString)[eval] {
    return eval(evalString);
}
```

It can't access anything outside of its scope, so it is a lot safer.  This includes access to `globalThis` and similar things, so it can't read cookies or make network requests either.

Also, we could do renaming

```javascript
export function assertNotCircular(lists) {
    lists.forEach(function(lists)[lists: outerLists, ...] {
        if (lists === outerLists) {
            throw new Error('Circular entry found');
        }
    })
}
```

`[lists: outerLists]` binds the `lists` where the function is defined to the identifier `outerLists` inside the body of the function.
This is similar to object destructuring syntax.

ALTERNATE: Instead of object destructuring semantics, if we used object creation semantics, it would be `[outerLists: lists]`.  We could then even do arbitrary calculations like `[outerLists: lists.filter(a => !!a)]`.

`[...]`, is the default, usually doesn't have to specified.  In this case we could explicitly bind `[Error]`, but we don't need to.

### Arrow function syntax
Placed after the arrow because otherwise it would be ambiguous.  I think.  I don't know, my brain isn't a parser.
```javascript
export const myFunction = (myParam) => [console]{
    console.log(myParam);
}
```

### Pure functions

A function that does not bind any closure is guaranteed to be pure.  Everyone loves this.

### Functions that are intended to be executed elsewhere.

For example, using the Puppeteer library we have the following
```javascript
const transformData = (data) => {
    return data.map(point => point * 2);
};
const result = await page.evaluate(() => {
    return window.myAppClient.doSomething(transformData(window.myAppClient.getData()));
});
```

This will break at runtime because the function being passed in is going to be stringified by Puppeteer and evaluated in a scope without access to `transformData`.

While
```javascript
page.evaluate(() => []{
    ...
}
```
would be analyzable by static tools (such as TypeScript) to find the issue ahead of time.

### Implicits

This is not a proposal for implicit parameters.  These values are bound at the time the function is declared, not at the callsite.

### Standard Library

No closure means no closure.  No access to `JSON.parse`, no access to `Math.random`, no access to `NaN` or `Symbol` or `Object`.

This will be unfortunate when trying to do something like
```javascript
export function factory(x)[] {
    return {a: x}
}
```
Because creating that object would require access to the `Object` class and its prototype.

Similarly, would this work? Would the engine be able to implicitly convert the `string` to a boxed `String` object in order to access the `.length` property?
```javascript
export function factory(x)[] {
    return `(${x})`.length;
}
```
Sayomg "no closure means no closure" might be fine but the alternative of always including the "Primitives" (`Object`, `Number`, `Boolean`, `Array`, `String`, `null`, `undefined`, `Symbol`, `BigInt`) is still being investigated.
`Map` and `Set` and friends would not need to be included, as there is no native syntax to produce those.