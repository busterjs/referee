# referee

A collection of assertions to be used with a unit testing framework. **referee** works well with any CommonJS compliant testing framework out of the box, and can easily be configured to work with most any testing framework. See also expectations if you like the alternative API
(`expect(thing).toBe*`).

**referee** contains lots of assertions. We strongly believe that high-level assertions are essential in the interest of producing clear and intent-revealing tests, and they also give you to-the-point failure messages.


## Assertions and refutations

Unlike most assertion libraries, **referee** does not have `assert.notXyz` assertions to refute some fact. Instead, it has *refutations*, heavily inspired by Ruby's [minitest](http://bfts.rubyforge.org/minitest/):

```js
var assert = referee.assert;
var refute = referee.refute;

assert.equals(42, 42);
refute.equals(42, 43);
```

Refutations help express "assert not ..." style verification in a much clearer way. It also brings with it a nice consistency in that any `assert.xyz` always has a corresponding `refute.xyz` that does the opposite check.

### `assert()`

```js
assert(actual[, message]);
```

Fails if `actual` is falsy (`0`, `""`, `null`, `undefined`, `NaN`). Fails with either the provided message or "Expected null to be truthy". This behavior differs from all other assertions, which prepend the optional message argument.

```js
assert({ not: "Falsy" }, "This will pass");
assert(null, "This will fail"); // Fails with custom message
assert(null); // Fails
assert(34);   // Passes
```

### `refute()`

```js
refute(actual[, message])
```

Fails if actual is truthy. Fails with either the provided message or "Expected null to be falsy". This behavior differs from all other refutations, which prepend the optional message argument.

```js
refute({ not: "Falsy" }, "This will fail"); // Fails with custom message
refute(null, "This will pass");
refute(null); // Passes
refute(34);   // Fails
```


## Predefined Assertions

The following assertions can be used with `assert` and `refute`. They
are described for `assert`, but the corresponding failure messages for
`refute` are also mentioned. For `refute` the behaviour is exactly
opposed.

All assertions support an optional `message` argument, which is
prepended to the failure message.

*Overview:*

* [`same()`](#same)
* [`equals()`](#equals)
* [`greater()`](#greater)
* [`less()`](#less)
* [`defined()`](#defined)
* [`isNull()`](#isnull)
* [`match()`](#match)
* [`isObject()`](#isobject)
* [`isFunction()`](#isfunction)
* [`isTrue()`](#istrue)
* [`isFalse()`](#isfalse)
* [`isString()`](#isstring)
* [`isBoolean()`](#isboolean)
* [`isNumber()`](#isnumber)
* [`isNaN()`](#isnan)
* [`isArray()`](#isarray)
* [`isArrayLike()`](#isarraylike)
* [`exception()`](#exception)
* [`near()`](#near)
* [`hasPrototype()`](#hasprototype)
* [`contains()`](#contains)
* [`tagName()`](#tagname)
* [`className()`](#classname)


### `same()`

```js
assert.same(actual, expected[, message])
```

Fails if `actual` is not the same object (`===`) as `expected`. To compare similar objects, such as { name: "Chris", id: 42 } and { id: 42, name: "Chris" } (not the same instance), see [`equals()`](#equals).

```js
var obj = { id: 42, name: "Chris" };
assert.same(obj, obj);                       // Passes
assert.same(obj, { id: 42, name: "Chris" }); // Fails
```

#### Messages

```js
assert.same.message = "${actual} expected to be the same object as ${expected}";
refute.same.message = "${actual} expected not to be the same object as ${expected}";
```


### `equals()`

```js
assert.equals(actual, expected[, message])
```

Compares `actual` to `expected` property by property. If the property count does not match, or if any of `actual`‘s properties do not match the corresponding property in `expected`, the assertion fails. Object properties are verified recursively.

If `actual` is `null` or `undefined`, an exact match is required. Date objects are compared by their `getTime` method. Regular expressions are compared by their string representations. Primitives are compared using `==`, i.e., with coercion.

`equals` passes when comparing an arguments object to an array if the both contain the same elements.

```js
assert.equals({ name: "Professor Chaos" }, { name: "Professor Chaos" }); // Passes
assert.equals({ name: "Professor Chaos" }, { name: "Dr Evil" }); // Fails
```

#### Messages

```js
assert.equals.message = "${actual} expected to be equal to ${expected}";
refute.equals.message = "${actual} expected not to be equal to ${expected}";
```


### `greater()`

```js
assert.greater(actual, expected[, message])
```

Fails if `actual` is equal to or less than `expected`.

```js
assert.greater(2, 1); // Passes
assert.greater(1, 1); // Fails
assert.greater(1, 2); // Fails
```

#### Messages

```js
assert.greater.message = "Expected ${actual} to be greater than ${expected}";
refute.greater.message = "Expected ${actual} to be less than or equal to ${expected}";
```


### `less()`

```js
assert.less(actual, expected[, message])
```

Fails if `actual` is equal to or greater than `expected`.

```js
assert.less(1, 2); // Passes
assert.less(1, 1); // Fails
assert.less(2, 1); // Fails
```

#### Messages

```js
assert.less.message = "Expected ${actual} to be less than ${expected}";
refute.less.message = "Expected ${actual} to be greater than or equal to ${expected}";
```


### `defined()`

```js
assert.defined(object[, message])
```

Fails if `object` is undefined.

```js
var a;
assert.defined({});  // Passes
assert.defined(a); // Fails
```

#### Messages

```js
assert.defined.message = "Expected to be defined";
refute.defined.message = "Expected ${actual} (${actualType}) not to be defined";
```


### `isNull()`

```js
assert.isNull(object[, message])
```

Fails if `object` is not `null`.

```js
assert.isNull(null, "This will pass");
assert.isNull({}, "This will fail");
assert.isNull(null); // Passes
assert.isNull({});   // Fails
```

#### Messages

```js
assert.isNull.message = "Expected ${actual} to be null";
refute.isNull.message = "Expected not to be null";
```


### `match()`

```js
assert.match(actual, matcher[, message])
```

Fails if `matcher` is not a partial match for `actual`. Accepts a wide range of input combinations. Note that `assert.match` is not symmetric - in some cases `assert.match(a, b)` may pass while `assert.match(b, a)` fails.

#### String matcher

In its simplest form, `assert.match` performs a case insensitive substring match. When the matcher is a string, the `actual` object is converted to a string, and the assertion passes if `actual` is a case-insensitive substring of expected as a string.

```js
assert.match("Give me something", "Give");                           // Passes
assert.match("Give me something", "sumptn");                         // Fails
assert.match({ toString: function () { return "yeah"; } }, "Yeah!"); // Passes
```

The last example is not symmetric. When the matcher is a string, the actual value is coerced to a string - in this case using `toString`. Changing the order of the arguments would cause the matcher to be an object, in which case different rules apply (see below).

#### Boolean matcher

Performs a strict (i.e. `===`) match with the object. So, only `true` matches `true`, and only `false` matches `false`.

#### Regular expression matcher

When the matcher is a regular expression, the assertion will pass if `expected.test(actual)` is `true`. `assert.match` is written in a generic way, so any object with a `test` method will be used as a matcher this way.

```js
assert.match("Give me something", /^[a-z\s]$/i); // Passes
assert.match("Give me something", /[0-9]/); // Fails
assert.match({ toString: function () { return "yeah!"; } }, /yeah/); // Passes
assert.match(234, /[a-z]/); // Fails
```

#### Number matcher

When the matcher is a number, the assertion will pass if `matcher == actual`.

```js
assert.match("123", 123); // Passes
assert.match("Give me something", 425); // Fails
assert.match({ toString: function () { return "42"; } }, 42); // Passes
assert.match(234, 1234); // Fails
```

#### Function matcher

When the matcher is a function, it is called with `actual` as its only argument. The assertion will pass if the function returns `true`. A strict match is performed against the return value, so a boolean `true` is required, truthy is not enough.

```js
// Passes
assert.match("123", function (exp) {
    return exp == "123";
});

// Fails
assert.match("Give me something", function () {
    return "ok";
});

// Passes
assert.match({
    toString: function () {
        return "42";
    }
}, function () { return true; });

// Fails
assert.match(234, function () {});
```

#### Object matcher

As mentioned above, if an object matcher defines a test method the assertion will pass if `matcher.test(actual)` returns truthy. If the object does not have a `test` method, a recursive match is performed. If all properties of `matcher` matches corresponding properties in `actual`, the assertion passes. Note that the object matcher does not care if the number of properties in the two objects are the same - only if all properties in the matcher recursively “matches” ones in the actual object.

```js
// Passes
assert.match("123", {
    test: function (arg) {
        return arg == 123;
    }
});

// Fails
assert.match({}, { prop: 42 });

// Passes
assert.match({
    name: "Chris",
    profession: "Programmer"
}, {
    name: "Chris"
});

// Fails
assert.match(234, {
    name: "Chris"
});
```

#### DOM elements

`assert.match` can be very helpful when asserting on DOM elements, because it allows you to compare several properties with one assertion:

```js
var el = document.getElementById("myEl");

assert.match(el, {
    tagName: "h2",
    className: "item",
    innerHTML: "Howdy"
});
```

#### Messages

```js
assert.match.exceptionMessage = "${exceptionMessage}";
refute.match.exceptionMessage = "${exceptionMessage}";
```

Used when the matcher function throws an exception. This happens if the matcher is not any of the accepted types, for instance, a boolean.

```js
assert.match.message = "${actual} expected to match ${expected}";
refute.match.message = "${actual} expected not to match ${expected}";
```


### `isObject()`

```js
assert.isObject(object[, message])
```

Fails if `object` is not an object or if it is `null`.

```js
assert.isObject({});             // Passes
assert.isObject(42);             // Fails
assert.isObject([1, 2, 3]);      // Passes
assert.isObject(function () {}); // Fails
```

#### Messages

```js
assert.isObject.message = "${actual} (${actualType}) expected to be object and not null";
refute.isObject.message = "${actual} expected to be null or not an object";
```


### `isFunction()`

```js
assert.isFunction(actual[, message])
```

Fails if `actual` is not a function.

```js
assert.isFunction({});             // Fails
assert.isFunction(42);             // Fails
assert.isFunction(function () {}); // Passes
```

#### Messages

```js
assert.isFunction.message = "${actual} (${actualType}) expected to be function";
refute.isFunction.message = "${actual} expected not to be function";
```


### `isTrue()`

```js
assert.isTrue(actual[, message])
```

Fails if `actual` is not `true`.

```js
assert.isTrue("2" == 2);  // Passes
assert.isTrue("2" === 2); // Fails
```

#### Messages

```js
assert.isTrue.message = "Expected ${actual} to be true";
refute.isTrue.message = "Expected ${actual} to not be true";
```


### `isFalse()`

```js
assert.isFalse(actual[, message])
```

Fails if `actual` is not `false`.

```js
assert.isFalse("2" === 2); // Passes
assert.isFalse("2" == 2);  // Fails
```

#### Messages

```js
assert.isFalse.message = "Expected ${actual} to be false";
refute.isFalse.message = "Expected ${actual} to not be false";
```


### `isString()`

```js
assert.isString(actual[, message])
```

Fails if the type of actual is not "string".

```js
assert.isString("2"); // Passes
assert.isString(2);   // Fails
```

#### Messages

```js
assert.isString.message = "Expected ${actual} (${actualType}) to be string";
refute.isString.message = "Expected ${actual} not to be string";
```


### `isBoolean()`

```js
assert.isBoolean(actual[, message])
```

Fails if the type of `actual` is not "boolean".

```js
assert.isBoolean(true);   // Passes
assert.isBoolean(2 < 2);  // Passes
assert.isBoolean("true"); // Fails
```

#### Messages

```js
assert.isBoolean.message = "Expected ${actual} (${actualType}) to be boolean";
refute.isBoolean.message = "Expected ${actual} not to be boolean";
```


### `isNumber()`

```js
assert.isNumber(actual[, message])
```

Fails if the type of `actual` is not "number" or is `NaN`.

```js
assert.isNumber(12);   // Passes
assert.isNumber("12"); // Fails
assert.isNumber(NaN);  // Fails
```

#### Messages

```js
assert.isNumber.message = "Expected ${actual} (${actualType}) to be a non-NaN number";
refute.isNumber.message = "Expected ${actual} to be NaN or a non-number value";
```


### `isNaN()`

```js
assert.isNaN(actual[, message])
```

Fails if `actual` is not `NaN`. Does not perform coercion in contrast to the standard javascript function `isNaN`.

```js
assert.isNaN(NaN);           // Passes
assert.isNaN("abc" / "def"); // Passes
assert.isNaN(12);            // Fails
assert.isNaN({});            // Fails, would pass for standard javascript function isNaN
```

#### Messages

```js
assert.isNaN.message = "Expected ${actual} to be NaN";
refute.isNaN.message = "Expected not to be NaN";
```


### `isArray()`

```js
assert.isArray(actual[, message])
```

Fails if the object type of `actual` is not `Array`.

```js
assert.isArray([1, 2, 3]); // Passes
assert.isArray({});        // Fails
```

#### Messages

```js
assert.isArray.message = "Expected ${actual} to be array";
refute.isArray.message = "Expected ${actual} not to be array";
```


### `isArrayLike()`

```js
assert.isArrayLike(actual[, message])
```

Fails if none of the following conditions are fulfilled:

* the object type of `actual` is `Array`
* `actual` is an `arguments object
* `actual` is an object providing a property `length` of type "number" and a `property` splice of type "function"

```js
assert.isArrayLike([1, 2, 3]);                            // Passes
assert.isArrayLike(arguments);                            // Passes
assert.isArrayLike({ length: 0, splice: function() {} }); // Passes
assert.isArrayLike({});                                   // Fails
```

#### Messages

```js
assert.isArrayLike.message = "Expected ${actual} to be array like";
refute.isArrayLike.message = "Expected ${actual} not to be array like";
```


### `keys()`

```js
assert.keys(object, keyArray[, message])
```

Fails if object’s own properties are not exactly the same as a given list.

```js
assert.keys({ test1: 't1', test2: 't2' }, ['test1']);                 // Fails - 'test2' is unexpected
assert.keys({ test1: 't1', test2: 't2' }, ['test1','test2','test3']); // Fails - 'test3' is not present
assert.keys({ test1: 't1', test2: 't2' }, ['test1','test2']);         // Passes
```

#### Messages

```js
assert.keys.message = "Expected ${actualObject} to have exact keys ${keys}";
refute.keys.message = "Expected not to have exact keys ${keys}";
```

### `exception()`

```js
assert.exception(callback[, matcher, message])
```

Fails if `callback` does not throw an exception. If the optional `matcher` is provided, the assertion fails if the callback either does not throw an exception, or if the exception does not meet the criterias of the given `matcher.

The `matcher` can be of type `object` or `function`. If the `matcher` is of type `object`, the captured error object and the `matcher` are passed to [`match()`](#match).

If the `matcher` is of type `function`, the captured error object is passed as argument to the `matcher` function, which has to return `true` for a matching error object, otherwise `false`.

```js
// Passes
assert.exception(function () {
    throw new Error("Ooops!");
});

// Fails
assert.exception(function () {});

// Passes
assert.exception(function () {
    throw new TypeError("Ooops!");
},  { name: "TypeError" });

// Fails, wrong exception type
assert.exception(function () {
    throw new Error("Aww");
}, { name: "TypeError" });

// Fails, wrong exception message
assert.exception(function () {
    throw new Error("Aww");
}, { message: "Ooops!" });

// Fails, wrong exception type
assert.exception(function () {
    throw new Error("Aww");
}, function (err) {
    if (err.name !== "TypeError") {
        return false;
    }
    return true;
}, "Type of exception is wrong!");  // with message to print, if test fails
```

#### Messages

```js
assert.exception.typeNoExceptionMessage = "Expected ${expected} but no exception was thrown";
assert.exception.message = "Expected exception";
assert.exception.typeFailMessage = "Expected ${expected} but threw ${actualExceptionType} (${actualExceptionMessage})\n${actualExceptionStack}";
assert.exception.matchFailMessage = "Expected thrown ${actualExceptionType} (${actualExceptionMessage}) to pass matcher function";
refute.exception.message = "Expected not to throw but threw ${actualExceptionType} (${actualExceptionMessage})";
```


### `near()`

```js
assert.near(actual, expected, delta[, message])
```

Fails if the difference between `actual` and `expected` is greater than `delta`.

```js
assert.near(10.3, 10, 0.5); // Passes
assert.near(10.5, 10, 0.5); // Passes
assert.near(10.6, 10, 0.5); // Fails
```

#### Messages

```js
assert.near.message = "Expected ${actual} to be equal to ${expected} +/- ${delta}";
refute.near.message = "Expected ${actual} not to be equal to ${expected} +/- ${delta}";
```

### `hasPrototype()`

```js
assert.hasPrototype(actual, prototype[, message])
```

Fails if `prototype` does not exist in the prototype chain of `actual`.

```js
assert.hasPrototype(function() {}, Function.prototype); // Passes
assert.hasPrototype(function() {}, Object.prototype);   // Passes
assert.hasPrototype({}, Function.prototype);            // Fails
```

#### Messages

```js
assert.hasPrototype.message = "Expected ${actual} to have ${expected} on its prototype chain";
refute.hasPrototype.message = "Expected ${actual} not to have ${expected} on its prototype chain";
```


### `contains()`

```js
assert.contains(haystack, needle[, message])
```

Fails if the array like object `haystack` does not contain the `needle` argument.

```js
assert.contains([1, 2, 3], 2);   // Passes
assert.contains([1, 2, 3], 4);   // Fails
assert.contains([1, 2, 3], "2"); // Fails
```

#### Messages

```js
assert.contains.message = "Expected [${actual}] to contain ${expected}";
refute.contains.message = "Expected [${actual}] not to contain ${expected}";
```


### `tagName()`

```js
assert.tagName(element, tagName[, message])
```

Fails if the `element` either does not specify a `tagName` property, or if its value is not a case-insensitive match with the expected `tagName`. Works with any object.

```js
assert.tagName(document.createElement("p"), "p"); // Passes
assert.tagName(document.createElement("h2"), "H2"); // Passes
assert.tagName(document.createElement("p"), "li");  // Fails
```

#### Messages

```js
assert.tagName.noTagNameMessage = "Expected ${actualElement} to have tagName property";
assert.tagName.message = "Expected tagName to be ${expected} but was ${actual}";
refute.tagName.noTagNameMessage = "Expected ${actualElement} to have tagName property";
refute.tagName.refuteMessage = "Expected tagName not to be ${actual}";
```


### `className()`

```js
assert.className(element, classNames[, message])
```

Fails if the `element` either does not specify a `className` property, or if its value is not a space-separated list of all class names in `classNames`.

`classNames` can be either a space-delimited string or an array of class names. Every class specified by classNames must be found in the object’s `className` property for the assertion to pass, but order does not matter.

```js
var el = document.createElement("p");
el.className = "feed item blog-post";

assert.className(el, "item");           // Passes
assert.className(el, "news");           // Fails
assert.className(el, "blog-post feed"); // Passes
assert.className(el, "feed items");     // Fails, "items" is not a match
assert.className(el, ["item", "feed"]); // Passes
```

#### Messages

```js
assert.className.noClassNameMessage = "Expected object to have className property";
assert.className.message = "Expected object's className to include ${expected} but was ${actual}";
refute.className.noClassNameMessage = "Expected object to have className property";
refute.className.message = "Expected object's className not to include ${expected}";
```

## Custom assertions

Custom, domain-specific assertions helps improve clarity and reveal intent in tests. They also facilitate much better feedback when they fail. You can add custom assertions that behave exactly like the built-in ones (i.e. with counting, message formatting, expectations and
more) by using the [`referee.add`](#refereeadd) method.

## Overriding assertion messages

The default assertion messages can be overridden. The messages to
overwrite are listed with each assertion. You can use the same keys for
string interpolation (e.g. `${actual}`, `${expected}`). equals:

```js
var assert = require('referee').assert;
assert.equals.message = "I wanted ${actual} == ${expected}!"

try {
    assert.equals(3, 4);
} catch (e) {
    console.log(e.message);
}

// Prints:
// "I wanted 3 == 4!"
```

## Events

`referee` is an event-emitter. Listen to events with `on`:

```js
referee.on("failure", function (err) {
    console.log(err.message);
});
```

### `pass` event

Signature:
```js
"pass", function () {}
```

Assertion passed. The callback is invoked with the assertion name, e.g.
`"equals"`, as its only argument. Note that this event is also emitted
when refutations pass.

### `failure` event

Signature:
```js
    "failure", function (error) {}
```

Assertion failed. The callback is invoked with an [`AssertionError`](#class-assertionerror) object.


## Stubs and spies

~The default Buster.JS bundle comes with built-in spies, stubs and mocks provided by Sinon.JS. The assertions are indisposable when working with spies and stubs. However, note that these assertions are technically provided by the integration package buster-sinon, not referee. This only matters if you use this package stand-alone.~

As for the normal assertions, the assertions for stubs and spies can be used with `assert` and `refute`. The description is for `assert`, but the corresponding failure messages for `refute` are also mentioned. For refute the behaviour is exactly opposite.

*Overview:*

* [`called()`](#called)
* [`callOrder()`](#callorder)
* [`calledOnce()`](#calledonce)
* [`calledTwice()`](#calledtwice)
* [`calledThrice()`](#calledthrice)
* [`calledOn()`](#calledon)
* [`alwaysCalledOn()`](#alwayscalledon)
* [`calledWith()`](#calledwith)
* [`alwaysCalledWith()`](alwayscalledwith)
* [`calledOnceWith()`](#calledoncewith)
* [`calledWithExactly()`](#calledwithexactly)
* [`alwaysCalledWithExactly()`](#alwayscalledwithexactly)
* [`threw()`](#threw)
* [`alwaysThrew()`](#alwaysthrew)

### `called()`

```js
assert.called(spy)
```

Fails if the `spy` has never been called.

```js
var spy = this.spy();

assert.called(spy); // Fails

spy();
assert.called(spy); // Passes

spy();
assert.called(spy); // Passes
```

#### Messages

```js
assert.called.message = "Expected ${0} to be called at least once but was never called";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
</dl>

```js
refute.called.message = "Expected ${0} to not be called but was called ${1}${2}";
```


<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The number of calls as a string. Ex: “two times”</dd>
    <dt>`${2}`:</dt>
    <dd>All calls formatted as a multi-line string</dd>
</dl>

### `callOrder()`

```js
assert.callOrder(spy, spy2, ...)
```

Fails if the spies were not called in the specified order.

```js
var spy1 = this.spy();
var spy2 = this.spy();
var spy3 = this.spy();

spy1();
spy2();
spy3();

assert.callOrder(spy1, spy3, spy2); // Fails
assert.callOrder(spy1, spy2, spy3); // Passes
```

#### Messages

```js
assert.callOrder.message = "Expected ${expected} to be called in order but were called as ${actual}";
refute.callOrder.message = "Expected ${expected} not to be called in order";
```

<dl>
    <dt>`${expected}`:</dt>
    <dd>A string representation of the expected call order</dd>
    <dt>`${actual}`:</dt>
    <dd>A string representation of the actual call order</dd>
</dl>


### `calledOnce()`

```js
assert.calledOnce(spy)
```

Fails if the `spy` has never been called or if it was called more than once.

```js
var spy = this.spy();

assert.calledOnce(spy); // Fails

spy();
assert.calledOnce(spy); // Passes

spy();
assert.calledOnce(spy); // Fails
```

#### Messages

```
assert.calledOnce.message = "Expected ${0} to be called once but was called ${1}${2}";
refute.calledOnce.message = "Expected ${0} to not be called exactly once${2}";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The number of calls, as a string. Ex: “two times”</dd>
    <dt>`${2}`:</dt>
    <dd>The call log. All calls as a string. Each line is one call and includes passed arguments, returned value and more</dd>
</dl>

### `calledTwice()`

```js
assert.calledTwice(spy)
```

Only passes if the `spy` was called exactly twice.

```js
var spy = this.spy();

assert.calledTwice(spy); // Fails

spy();
assert.calledTwice(spy); // Fails

spy();
assert.calledTwice(spy); // Passes

spy();
assert.calledTwice(spy); // Fails
```

#### Messages

```js
assert.calledTwice.message = "Expected ${0} to be called twice but was called ${1}${2}";
refute.calledTwice.message = "Expected ${0} to not be called exactly twice${2}";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The number of calls, as a string. Ex: “two times”</dd>
    <dt>`${2}`:</dt>
    <dd>The call log. All calls as a string. Each line is one call and includes passed arguments, returned value and more</dd>
</dl>

### `calledThrice()`

```js
assert.calledThrice(spy)
```

Only passes if the `spy` has been called exactly three times.

```js
var spy = this.spy();

assert.calledThrice(spy); // Fails

spy();
assert.calledThrice(spy); // Fails

spy();
assert.calledThrice(spy); // Fails

spy();
assert.calledThrice(spy); // Passes

spy();
assert.calledThrice(spy); // Fails
```

#### Messages

```js
assert.calledThrice.message = "Expected ${0} to be called thrice but was called ${1}${2}";
refute.calledThrice.message = "Expected ${0} to not be called exactly thrice${2}";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The number of calls, as a string. Ex: “two times”</dd>
    <dt>`${2}`:</dt>
    <dd>The call log. All calls as a string. Each line is one call and includes passed arguments, returned value and more</dd>
</dl>

### `calledOn()`

```js
assert.calledOn(spy, obj)
```

Passes if the `spy` was called at least once with `obj` as its `this` value.

```js
var spy = this.spy();
var obj1 = {};
var obj2 = {};
var obj3 = {};

spy.call(obj2);
spy.call(obj3);

assert.calledOn(spy, obj1); // Fails
assert.calledOn(spy, obj2); // Passes
assert.calledOn(spy, obj3); // Passes
```

#### Messages

```js
assert.calledOn.message = "Expected ${0} to be called with ${1} as this but was called on ${2}";
refute.calledOn.message = "Expected ${0} not to be called with ${1} as this";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The object obj which is expected to have been this at least once</dd>
    <dt>`${2}`:</dt>
    <dd>List of objects which actually have been `this`</dd>
</dl>

### `alwaysCalledOn()`

```js
assert.alwaysCalledOn(spy, obj)
```

Passes if the `spy` was always called with `obj` as its `this` value.

```js
var spy1 = this.spy();
var spy2 = this.spy();
var obj1 = {};
var obj2 = {};

spy1.call(obj1);
spy1.call(obj2);

spy2.call(obj2);
spy2.call(obj2);

assert.alwaysCalledOn(spy1, obj1); // Fails
assert.alwaysCalledOn(spy1, obj2); // Fails
assert.alwaysCalledOn(spy2, obj1); // Fails
assert.alwaysCalledOn(spy2, obj2); // Passes
```

#### Messages

```js
assert.alwaysCalledOn.message = "Expected ${0} to always be called with ${1} as this but was called on ${2}";
refute.alwaysCalledOn.message = "Expected ${0} not to always be called with ${1} as this";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The object obj which is expected always to have been `this`</dd>
    <dt>`${2}`:</dt>
    <dd>List of objects which actually have been `this`</dd>
</dl>


### `calledWith()`

```js
assert.calledWith(spy, arg1, arg2, ...)
```

Passes if the `spy` was called at least once with the specified arguments. Other arguments may have been passed after the specified ones.

```js
var spy = this.spy();
var arr = [1, 2, 3];
spy(12);
spy(42, 13);
spy("Hey", arr, 2);

assert.calledWith(spy, 12);         // Passes
assert.calledWith(spy, "Hey");      // Passes
assert.calledWith(spy, "Hey", 12);  // Fails
assert.calledWith(spy, "Hey", arr); // Passes
```

#### Messages

```js
assert.calledWith.message = "Expected ${0} to be called with arguments ${1}${2}";
refute.calledWith.message = "Expected ${0} not to be called with arguments ${1}${2}";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The expected arguments</dd>
    <dt>`${2}`:</dt>
    <dd>String representation of all calls</dd>
</dl>

### `alwaysCalledWith()`

```js
assert.alwaysCalledWith(spy, arg1, arg2, ...)
```

Passes if the `spy` was always called with the specified arguments. Other arguments may have been passed after the specified ones.

```js
var spy = this.spy();
var arr = [1, 2, 3];
spy("Hey", arr, 12);
spy("Hey", arr, 13);

assert.alwaysCalledWith(spy, "Hey");          // Passes
assert.alwaysCalledWith(spy, "Hey", arr);     // Passes
assert.alwaysCalledWith(spy, "Hey", arr, 12); // Fails
```

#### Messages

```js
assert.alwaysCalledWith.message = "Expected ${0} to always be called with arguments ${1}${2}";
refute.alwaysCalledWith.message = "Expected ${0} not to always be called with arguments${1}${2}";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The expected arguments</dd>
    <dt>`${2}`:</dt>
    <dd>String representation of all calls</dd>
</dl>

### `calledOnceWith()`

```js
assert.calledOnceWith(spy, arg1, arg2, ...)
```

Passes if the `spy` was called exactly once and with the specified arguments. Other arguments may have been passed after the specified ones.

```js
var spy = this.spy();
var arr = [1, 2, 3];
spy(12);

assert.calledOnceWith(spy, 12);     // Passes
assert.calledOnceWith(spy, 42);     // Fails

spy(42, 13);
assert.calledOnceWith(spy, 42, 13); // Fails
```

#### Messages

```js
assert.calledOnceWith.message = "Expected ${0} to be called once with arguments ${1}${2}";
refute.calledOnceWith.message = "Expected ${0} not to be called once with arguments ${1}${2}";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The expected arguments</dd>
    <dt>`${2}`:</dt>
    <dd>String representation of all calls</dd>
</dl>

### `calledWithExactly()`

```js
assert.calledWithExactly(spy, arg1, arg2, ...)
```

Passes if the `spy` was called at least once with exactly the arguments specified.

```js
var spy = this.spy();
var arr = [1, 2, 3];
spy("Hey", arr, 12);
spy("Hey", arr, 13);

assert.calledWithExactly(spy, "Hey", arr, 12); // Passes
assert.calledWithExactly(spy, "Hey", arr, 13); // Passes
assert.calledWithExactly(spy, "Hey", arr);     // Fails
assert.calledWithExactly(spy, "Hey");          // Fails
```

#### Messages

```js
assert.calledWithExactly.message = "Expected ${0} to be called with exact arguments ${1}${2}";
refute.calledWithExactly.message = "Expected ${0} not to be called with exact arguments${1}${2}";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The expected arguments</dd>
    <dt>`${2}`:</dt>
    <dd>String representation of all calls</dd>
</dl>

### `alwaysCalledWithExactly()`

```js
assert.alwaysCalledWithExactly(spy, arg1, arg2, ...)
```

Passes if the `spy` was always called with exactly the arguments specified.

```js
var spy = this.spy();
var arr = [1, 2, 3];
spy("Hey", arr, 12);

assert.alwaysCalledWithExactly(spy, "Hey", arr, 12); // Passes
assert.alwaysCalledWithExactly(spy, "Hey", arr);     // Fails
assert.alwaysCalledWithExactly(spy, "Hey");          // Fails

spy("Hey", arr, 13);
assert.alwaysCalledWithExactly(spy, "Hey", arr, 12); // Fails
```

#### Messages

```js
assert.alwaysCalledWithExactly.message = "Expected ${0} to always be called with exact arguments ${1}${2}";
refute.alwaysCalledWithExactly.message = "Expected ${0} not to always be called with exact arguments${1}${2}";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The expected arguments</dd>
    <dt>`${2}`:</dt>
    <dd>String representation of all calls</dd>
</dl>


### `threw()`

```js
assert.threw(spy[, exception])
```

Passes if the `spy` threw at least once the specified `exception`. The `exception` can be a string denoting its type, or an actual object. If `exception` is not specified, the assertion passes if the `spy` ever threw any exception.

```js
var exception1 = new TypeError();
var exception2 = new TypeError();
var exception3 = new TypeError();
var spy = this.spy(function(exception) {
    throw exception;
});

function callAndCatchException(spy, exception) {
    try {
        spy(exception);
    } catch(e) {
    }
}

callAndCatchException(spy, exception1);
callAndCatchException(spy, exception2);

assert.threw(spy); // Passes
assert.threw(spy, “TypeError”); // Passes
assert.threw(spy, exception1); // Passes
assert.threw(spy, exception2); // Passes
assert.threw(spy, exception3); // Fails

callAndCatchException(spy, exception3); assert.threw(spy, exception3); // Passes
```

#### Messages

```js
assert.threw.message = "Expected ${0} to throw an exception${1}";
refute.threw.message = "Expected ${0} not to throw an exception${1}";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The expected exception</dd>
</dl>

### `alwaysThrew()`

```js
assert.alwaysThrew(spy[, exception])
```

Passes if the `spy` always threw the specified `exception`. The `exception` can be a string denoting its type, or an actual object. If `exception` is not specified, the assertion passes if the `spy` ever threw any exception.

```js
var exception1 = new TypeError();
var exception2 = new TypeError();
var spy = this.spy(function(exception) {
    throw exception;
});

function callAndCatchException(spy, exception) {
    try {
        spy(exception);
    } catch(e) {

    }
}

callAndCatchException(spy, exception1);
assert.alwaysThrew(spy); // Passes
assert.alwaysThrew(spy, “TypeError”); // Passes
assert.alwaysThrew(spy, exception1); // Passes

callAndCatchException(spy, exception2);
assert.alwaysThrew(spy); // Passes
assert.alwaysThrew(spy, “TypeError”); // Passes
assert.alwaysThrew(spy, exception1); // Fails
```

#### Messages

```js
assert.alwaysThrew.message = "Expected ${0} to always throw an exception${1}";
refute.alwaysThrew.message = "Expected ${0} not to always throw an exception${1}";
```

<dl>
    <dt>`${0}`:</dt>
    <dd>The spy</dd>
    <dt>`${1}`:</dt>
    <dd>The expected exception</dd>
</dl>

## Expectations

All of referee's assertions and refutations are also exposed as "expectations". Expectations is just a slightly different front-end to the same functionality, often preferred by the BDD inclined.

Expectations mirror assertions under different names. Refutations can be expressed using `expect(obj).not` and then calling either of the expectations on the resulting object.

```js
var expect = require('referee').expect;

expect({ id: 42 }).toBeObject(); // Passes
expect("Somewhere in here").toMatch("in"); // Passes
expect(42).not.toEqual(43); // Passes
```

### `expect.toBe()`

```js
expect(actual).toBe(expected)
```

See [`same()`](#same)

### `expect.toEqual()`

```js
expect(actual).toEqual(expected)
```

See [`equals()`](#equals)

### `expect.toBeGreaterThan()`

```js
expect(actual).toBeGreaterThan(expected)
```

See [`greater()`](#greater)

### `expect.toBeLessThan()`

```js
expect(actual).toBeLessThan(expected)
```

See [`less()`](#less)

### `expect.toBeDefined()`

```js
expect(actual).toBeDefined(expected)
```

See [`defined()`](#defined)

### `expect.toBeNull()`

```js
expect(actual).toBeNull(expected)
```

See [`isNull()`](#isnull)

### `expect.toMatch()`

```js
expect(actual).toMatch(expected)
```

See [`match()`](#match)

### `expect.toBeObject()``

```js
expect(actual).toBeObject(expected)
```

See [`isObject()`](#isobject)

### `expect.toBeFunction()`

```js
expect(actual).toBeFunction(expected)
```

See [`isFunction()`](#isfunction)

### `expect.toBeTrue()`

```js
expect(actual).toBeTrue()
```

See [`isTrue()`](#istrue)

### `expect.toBeFalse()`

```js
expect(actual).toBeFalse()
```

See [`isFalse()`](#isfalse)

### `expect.toBeString()`

```js
expect(actual).toBeString()
```

See [`isString()`](#isstring)

### `expect.toBeBoolean()`

```js
expect(actual).toBeBoolean()
```

See [`isBoolean()`](#isboolean)

### `expect.toBeNumber()`

```js
expect(actual).toBeNumber()
```

See [`isNumber()`](#isnumber)

### `expect.toBeNaN()`

```js
expect(actual).toBeNaN()
```

See [`isNaN()`](#isnan)

### `expect.toBeArray()`

```js
expect(actual).toBeArray()
```

See [`isArray()`](#isarray)

### `expect.toBeArrayLike()`

```js
expect(actual).toBeArrayLike()
```

See [`isArrayLike()`](#isarraylike)

### `expect.toHaveKeys()`

```js
expect(object).toHaveKeys(keyArray)
```

See [`keys()`](#keys)

### `expect.toThrow()`

```js
expect(actual).toThrow(expected)
```

See [`exception()`](#exception)

### `expect.toBeNear()`

```js
expect(actual).toBeNear(expected, delta)
```

See [`near()`](#near)

### `expect.toHavePrototype()`

```js
expect(actual).toHavePrototype(prototype)
```

See [`hasPrototype()`](#hasprototype)

### `expect.toContain()`

```js
expect(haystack).toContain(needle)
```

See [`contains()`](#contains)

### `expect.toHaveTagName()`

```js
expect(actual).toHaveTagName(expected)
```

See [`tagName()`](#tagname)

### `expect.toHaveClassName()`

```js
expect(actual).toHaveClassName(expected)
```

See [`className()`](#classname)

### `expect.toHaveBeenCalled()`

```js
expect(spy).toHaveBeenCalled()
```

See [`called()`](#called)

### `expect.toHaveBeenCalledOnce()`

```js
expect(spy).toHaveBeenCalledOnce(expected)
```

See [`calledOnce()`](calledonce)

### `expect.toHaveBeenCalledTwice()`

```
expect(spy).toHaveBeenCalledTwice(expected)
```

See [`calledTwice()`](#calledtwice)

### `expect.toHaveBeenCalledThrice()`

```js
expect(spy).toHaveBeenCalledThrice(expected)
```

See [`calledThrice()`](#calledthrice)

### `expect.toHaveBeenCalledWith()`

```js
expect(spy).toHaveBeenCalledWith(arg1, arg2, ...)
```

See [`calledWith()`](#calledwith)

### `expect.toHaveBeenCalledOnceWith()`

```js
expect(spy).toHaveBeenCalledOnceWith(arg1, arg2, ...)
```

See [`calledOnceWith()`](#calledoncewith)

## Methods

### `referee.fail()`

```js
referee.fail(message)
```

When an assertion fails, it calls `referee.fail()` with the failure message as the only argument. The built-in `fail` function both throws an [`AssertionError()`](#class-assertionerror) and emits it to the `failure` event. The error can be caught and handled by the test runner. If this behavior is not suitable for your testing framework of choice, you can override `referee.fail()` to make it do the right thing.

Example: To use `referee with JsTestDriver, you can simply configure it as follows:

```js
referee.fail = function (message) {
    fail(message);
};
```

Where the global `fail` function is the one provided by JsTestDriver.

It is possible to make the default `assert.fail` method only emit an event and not throw an error. This may be suitable in asynchronous test runners, where you might not be able to catch exceptions. To silence exceptions, see the `throwOnFailure` property.


### `referee.format()`

```js
referee.format(object)
```

Values inserted into assertion messages using the `${n}` switches are formatted using `referee.format()`. By default this method simply coerces the object to a string.

A more expressive option is to use `buster-format`, which is a generic function for formatting objects nicely as ASCII. For nice ASCII formatting of objects (including DOM elements) do:

```js
referee.format = buster.format.ascii;
```


### `referee.add()`

```js
referee.add(name, options)
```

Add a custom assertion. Using this ‘macro’ to add project specific assertions has a few advantages:

* Assertions will be counted
* Failure messages will have interpolated arguments formatted by `referee.format()`
* A single function generates both an assertion and a refutation
* If using expectations, an expectation can easily be generated as well
* When `failOnNoAssertions` is set to `true`, the assertion will behave correctly (may be important for asynchronous tests)
* The assertion will fail if too few arguments are passed

Here’s an example of adding a “foo” assertion, that only passes when its only argument is the string “foo”:

```js
var assert = referee.assert;
var refute = referee.refute;
var expect = referee.expect;

referee.add("isFoo", {
    assert: function (actual) {
        return actual == "foo";
    },
    assertMessage: "Expected ${0} to be foo!",
    refuteMessage: "Expected not to be foo!",
    expectation: "toBeFoo"
});

// Now you can do:
// Passes
assert.isFoo("foo");

// Fails: "[assert.isFoo] Expected { id: 42 } to be foo!"
assert.isFoo({ id: 42 });

// Fails: "[refute.isFoo] Expected not to be foo!"
refute.isFoo("foo");

// Passes
expect("foo").toBeFoo();

// To support custom messages, do this:
referee.add("isFoo", {
    assert: function (actual) {
        return actual == "foo";
    },
    assertMessage: "${1}Expected ${0} to be foo!",
    refuteMessage: "${1}Expected not to be foo!",
    expectation: "toBeFoo",
    values: function (thing, message) {
        return [thing, message ? message + " " : ""];
    }
});

// Fails: "[assert.isFoo] Ouch: Expected { id: 42 } to be foo!"
assert.isFoo({ id: 42 }, "Ouch");
```

#### Error message value interpolation

Arguments are available in assertion failure messages using the `"${n}"` switches, where `n` is a number. You can also use named variables by setting properties on `this` in the assertion/refutation function:

```js
referee.add("isString", {
    assert: function (actual) {
        this.actualType = typeof actual;
        return this.actualType == "string";
    },
    assertMessage: "Expected ${0} (${actualType}) to be string",
    refuteMessage: "Expected not to be string",
    expectation: "toBeString"
});
```

##### Arguments

<dl>
    <dt>`name`:</dt>
    <dd>The name of the new assertion/refutation</dd>
    <dt>options</dt>
    <dd>
        <dl>
            <dt>`assert`:</dt>
            <dd>
The verification function. Should return `true` when the assertion passes. The generated refutation will pass when the function returns `false`.

In some cases the refutation may not be the exact opposite of the assertion. If that is the case you should provide `options.refute` for the custom refutation.

The number of formal parameters the function accepts determines the number of required arguments to the function. If the assertion is called with less arguments than expected, referee will fail it before your custom function is even called.

All arguments are available for interpolation into the resulting error message. The first argument will be available as `"${0}"`, the second as `"${1}"` and so on. If you want to embed other values than exact arguments into the string, you can set properties on this in the custom assertion, and refer to them as `"${name}"` in the message.
            </dd>
            <dt>`refute`:</dt>
            <dd>Custom refutation function. Used over `!assert()` if provided.</dd>
            <dt>`assertMessage`:</dt>
            <dd>The error message to use when the assertion fails. The message may refer to arguments through switches like `"${0}"`` and so on (see above, under the assert argument). The message is exposed on the generated assertion as the property `assert.[name].message`.</dd>
            <dt>`refuteMessage`:</dt>
            <dd>Like `assertMessage`, but for refutations. Exposed as `refute.[name].message`.</dd>
            <dt>`values:`</dt>
            <dd>A function that maps values to be interpolated into the failure messages. This can be used when you need something more/else than the actual `arguments` in order.</dd>
            <dt>expectation:</dt>
            <dd>The name of the assertion as an expectation, e.g. “toBeSomething”. Optional.</dd>
        </dl>
    </dd>
</dl>

## Properties

### `referee.count`

`Number` increasing from 0.

`referee.count` is incremented anytime an assertion is called. The assertion counter can be reset to any number at your convenience.

### `referee.throwOnFailure

`Boolean`.

When using the default `referee.fail()` implementation, this property can be set to `false` to make assertion failures not throw exceptions (i.e. only emit events). This may be suitable in asynchronous test runners, where you might not be able to catch exceptions.

## Supporting objects

### `class AssertionError()`

An exception (specifically, an `Error` object) whose `name` property is `"AssertionError"`.