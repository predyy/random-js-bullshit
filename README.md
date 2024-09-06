# Random JavaScript bullshit

JavaScript was created by Brendan Eich in **10 days** in 1995.
Thats probably the reason why it is used in web development, server-side development, mobile app development, and even in machine learning today.

This document contains some of the weird and fun things I ran into while working with JavaScript over the years.

## Types and Type Coercions

### Types

```javascript
// Type of Not a Number is number
typeof NaN; // "number"

// Type of null is object
typeof null; // "object"

// Type of array is object
typeof []; // "object"
// But
arr instanceof Array; // true
```

### Comparing stuff

#### Equality is not transitive

```javascript
[] == 0; // true
0 == "0"; // true
"0" == []; // false
```

#### Arrays are truthy except when they are not

An empty array is considered truthy when evaluated in a boolean context, but it loosely equals false.

```javascript
// ![] is boolean negation. Since an empty array is a truthy value, ![] becomes false
// [] == false because [] is converted to 0 and false is also converted to 0
[] == ![]; // true

// !![] is double negation. Since an empty array is a truthy value, !![] becomes true
!![]; // true

[] == false // true
```

#### `null` and `undefined` are only equal to themselves and each other

```javascript
// null and undefined are both falsy values
null == undefined; // true
// but you can still use === to compare them
null === undefined; // false
```

#### `null` is greater than or equal to 0, and less than or equal to 0 but not equal to 0

When using comparison operators like `>` and `<` JavaScript converts `null` to `0`. Therefore, `0 > 0` and `0 < 0` are both `false`, but `0 >= 0` and `0 <= 0` are `true`.

```javascript
// Some fun comparisons with null
null == 0;  // false
0 > null;   // false
0 < null;   // false
0 >= null;  // true
0 <= null;  // true
```

#### `NaN` is not equal to anything, not even itself

```javascript
// This is a bit weird
NaN == NaN; // false
// This surely won't happen when using ===
NaN === NaN; // false
// Nope. Just use isNaN
```

#### number objects are not numbers
  
```javascript
42 == new Number(42)  // true
42 === new Number(42) // false

// Strict comparison fails because one is a primitive (42) and the other is an object (new Number(42)).
```

#### Some more fun with equality

```javascript
// false is coerced to 0 and "0" is coerced to 0
false == "0"; // true
// empty array is coerced to "" and that is coerced to 0
false == []; // true
// empty string is coerced to 0 because Number("") is 0
"" == 0; // true
```

### Math operations with type coercion

Javascript is a bit weird when it comes to math operations, it will try to convert the operands to the same type and then add them.
This will either result in string concatenation or numerical addition seemingly at random.

#### Arrays and objects

This happens because first `{} `is interpreted as an empty code block. So the expression becomes `+{}`, and since empty object will be cast as `"[object Object]"` this becomes `+"[object Object]"`. Which is `NaN`. Technically true, the best kind of true
```javascript
{} + {} // NaN
```

This happens bacause array is converted to string when adding with `+` operator.
So `["foo"] + ["bar1", "bar2"]` becomes `"foo" + "bar1,bar2"` which is `"foobar1,bar2".
```javascript
[] + [] // ""
```

In this case, the array is converted to string and then the object is converted to string
```javascript
[] + {} // "[object Object]"
```

This one is a bit weird. `{}` is interpreted as an empty code block and this technically becomes `+[]`. 
Which then cast to empty string and that is then coerced to `0` using the unary `+` operator, because of course it is.
```javascript
{} + [] // 0
```

#### Numbers and strings

```javascript
// Mixing numbers and strings is fun
console.log(1 + "2");     // "12" (string concatenation)
console.log("2" + 1);     // "21" (string concatenation)
console.log(1 + 2 + "3"); // "33" (1 + 2 = 3, then "3" is concatenated)

console.log(1 + "2");    // "12" (string concatenation)
console.log(1 - "2");    // -1 (arithmetic subtraction)
console.log("10" - 2);   // 8 (arithmetic subtraction)
console.log("10" + 2);   // "102" (string concatenation)
```

Because `NaN` is a number, it can be used in math operation as a number and compared to other numbers. But it will always return `false`.

```javascript
NaN - 1000 < 100;   // false
NaN - 1000 > 100;   // false
NaN - 1000 == 100;  // false
```

#### Booleans

JavaScript treats booleans like numbers in arithmetic operations.

```javascript
console.log(true + true);   // 2
console.log(true - false);  // 1
console.log(true * false);  // 0
```

#### Infinity

We can divide by zero in JavaScript, and the result is `Infinity`.

```javascript
console.log(Infinity + 1); // Infinity
console.log(Infinity - Infinity); // NaN
console.log(Infinity * 0); // NaN
console.log(1 / 0); // Infinity, even though 1 / 0 is mathematically undefined and JS has a NaN value
```

### Some more fun with types

```javascript
// This is perfectly valid JavaScript code 
console.log("5" + - + - - + - - + + - + - + - - - "-2") // "5-2"
console.log("5" + - + - - + - - + + - + - + - + - - - "-2") // "52"
```

## Built-in Functions

### Many things are NaN

This happens because isNaN will try to convert the argument to a number before checking if it is NaN. As a result, strings and other values that can’t be converted to numbers will return true because the conversion yields NaN.

```javascript
// These are all NaN
isNaN("NaN"); // true
isNaN(undefined); // true
isNaN({}); // true

// But not these
isNaN(123); // false
isNaN("123"); // false
isNaN([]); // false
isNaN(null); // false
```

### Parsing ints can be fun

`parseInt` will convert armunet to string before parsing it.

```javascript
// parseInt(0.5) becomes parseInt("0.5") which is 0
// But parseInt(0.0000005) becomes parseInt("5e-7") which is 5

parseInt(0.05); // 0
parseInt(0.005); // 0
parseInt(0.0005); // 0
parseInt(0.000005); // 0
parseInt(0.0000005); // 5
```

For legacy reasons, parseInt will treat numbers with leading 0 as octal

```javascript
parseInt(08) // 8
parseInt(09) // 9
parseInt(010) // 8
parseInt(011) // 9
```

### Sorting array of numbers sorts them alphabetically... as strings

```javascript
const numbers = [10, 2, 30, 21];
numbers.sort();
console.log(numbers); // [10, 2, 21, 30] 

// To sort numbers correctly, you need to provide a comparison function
```

Also if array contains `undefined` it will be treated as largest possible value.

```javascript
const arr = [3, 5, undefined, 1];
arr.sort();
console.log(arr);  // [1, 3, 5, undefined]
```

### `Math.max` and `Math.min` 

```javascript
console.log(Math.max());  // -Infinity
console.log(Math.min());  // Infinity
```
## Arrays

You can manually set the length of an array. And you can also set elements outside the bounds of the array. This will create empty slots in the array. Length property doesn't count elements in array but the highest index + 1.

### Indexes and emply slots

```javascript
const arr = [1, 2, 3];
arr.length = 5;
console.log(arr); // [1, 2, 3, <2 empty slots>]
console.log(arr.length); // 5

arr[10] = "ten";
console.log(arr.length); // 11
console.log(arr); // [1, 2, 3, <7 empty slots>, 'ten']
```

Of course you can use negative numbers as array indices in JavaScript, but it won't behave the way you might expect.

```javascript
const arr = [1, 2, 3];
arr[-1] = "negative";
console.log(arr[-1]); // "negative"
console.log(arr.length); // 3
console.log(arr); // [1, 2, 3] (but arr[-1] exists as a property)
```

JavaScript allows trailing commas in arrays, which can lead to weird behavior with length.

```javascript
const arr = [1, 2, 3, ,];
console.log(arr.length); // 4
console.log(arr); // [1, 2, 3, empty]
```

### Deleting elements

`delete` function behaves differently when used on arrays instead of objects. When used on arrays, it will remove the element but leave an empty slot in the array. Even though array are objects, they are a special kind of object, because reasons.

```javascript
const arr = [1, 2, 3];
delete arr[1];
console.log(arr); // [1, empty, 3]
console.log(arr.length); // 3

const obj = { a: 1, b: 2 };
delete obj.b;
console.log(obj); // {a: 1}
```

### Arrays as keys in objects

Arrays can be used as keys in objects. No really, they can. This is because arrays they are coerced to strings when used as keys.

```javascript
const obj = {};
obj[[1, 2, 3]] = "value";
console.log(obj); // { "1,2,3": "value" }
console.log(obj["1,2,3"]); // "value"
```

### The `for...in` loop over arrays

The for...in loop is typically used to iterate over object properties, but it can also be used on arrays, though it might not behave as expected.

```javascript
const arr = [10, 20, 30];
arr.foo = "bar";

for (let i in arr) {
  console.log(i);  // "0", "1", "2", "foo"
}
```

`for...in` iterates over all properties of an object, including array elements and custom properties. This is why foo shows up in the loop. It's better to use `for...of` iterate over array values.

## Functions

### Functions arguments

Even though you can define a function with a certain number of arguments, you can call it with more or fewer arguments. The extra arguments can be accessed using the `arguments` object and it will work fine.

```javascript
function test(a, b) {
  console.log(arguments[0]); // 1
  console.log(arguments[1]); // 2
  console.log(arguments[2]); // 3 (even though there's no third parameter in the function definition)
}

test(1, 2, 3);
```

But arrow functions don't have the `arguments` object. Instead they inherit the `arguments` from their surrounding context.

```javascript
function foo() {
  const arrow = () => {
    console.log(arguments[0]);
		console.log(arguments[1]);
		console.log(arguments[2]);
  };
  arrow(1, 2, 3);
}

foo(4, 5, 6); // 4, 5, 6
```

### Function hoisting wierdness

In JavaScript, function declarations are hoisted to the top of the scope. This means that you can call a function before it is declared in the code. However, function expressions (where a function is assigned to a variable) are not hoisted the same way.

```javascript
hoistedFunction(); // Works fine

function hoistedFunction() {
  console.log("This function is hoisted");
}

// Unhoisted function expression will throw an error
try {
  unhoistedFunction(); // Throws error: unhoistedFunction is not a function
} catch (error) {
  console.log(error);
}

var unhoistedFunction = function () {
  console.log("This function is not hoisted");
};
```

### Functions are objects and have constructor

You can create functions dynamically using the `Function` constructor, which works like `eval` and is considered bad practice!
  
```javascript
const add = new Function('a', 'b', 'return a + b');
console.log(add(2, 3));  // 5
```

## Document is undefined, or is it?

`document.all` has a very peculiar behavior. Even though it's an object, JavaScript treats it as `undefined` in some contexts due to its legacy usage. Modern browsers implement this behavior for backward compatibility with old websites that relied on this.

```javascript
a = document.all;
typeof a === "undefined" && a == undefined && a !== undefined; // true
```
