[](https://stackoverflow.com/posts/3390426/timeline)

If you are interested in finding out whether a variable has been declared regardless of its value, then using the `in` operator is the safest way to go. Consider this example:

```javascript
// global scope
var theFu; // theFu has been declared, but its value is undefined
typeof theFu; // "undefined"
```

But this may not be the intended result for some cases, since the variable or property was declared but just not initialized. Use the `in` operator for a more robust check.

```javascript
"theFu" in window; // true
"theFoo" in window; // false
```

If you are interested in knowing whether the variable hasn't been declared or has the value `undefined`, then use the `typeof` operator, which is guaranteed to return a string:

```javascript
if (typeof myVar !== 'undefined')
```

Direct comparisons against `undefined` are troublesome as `undefined` can be overwritten.

```javascript
window.undefined = "foo";
"foo" == undefined // true
```

As @CMS pointed out, this has been patched in ECMAScript 5th ed., and `undefined` is non-writable.

`if (window.myVar)` will also include these falsy values, so it's not very robust:

false
0
""
NaN
null
undefined

Thanks to @CMS for pointing out that your third case - `if (myVariable)` can also throw an error in two cases. The first is when the variable hasn't been defined which throws a `ReferenceError`.

```javascript
// abc was never declared.
if (abc) {
    // ReferenceError: abc is not defined
} 
```

The other case is when the variable has been defined, but has a getter function which throws an error when invoked. For example,

```javascript
// or it's a property that can throw an error
Object.defineProperty(window, "myVariable", { 
    get: function() { throw new Error("W00t?"); }, 
    set: undefined 
});
if (myVariable) {
    // Error: W00t?
}
```