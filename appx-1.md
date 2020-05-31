## Type class polymorphism through dictionary passing style

Typescript does not natively support Haskell-style implicit type classes, but simple name overloading. While there is a way to imitate the former, it leads to rather entangled and complex type definitions. For the sake of simplicity I decided to fall back to dictionary passing style, which can be considered the poor man's type classes.

With DPS we must pass each overloaded operation that an ad-hoc polymorphic function requires as an argument. If a function requires more than one overloaded operation it defines a type dictionary as its first formal parameter. A type dictionary is nothing more than a plain old Javascript `Object` containing the overloaded operations as properties. This way we are freed from a specific argument order. If an ad-hoc polymorphic function only requires a single overloaded name, we simply pass the bare operation:

```javascript
foo({f, g, x}) (arg1) (arg2);
//  ^^^^^^^^^ type dictionary
//              ^^^^   ^^^^ regular arguments

bar(f) (arg1) (arg2);
//  ^ bare overloaded operation
```
But how we organize the overloaded operations themselves? Well, we keep things simple and just use a name prefix, which is essentially an abbrevation of the corresponding type, in order to avoid name clashes:

```javascript
const arrMap = f => xs => /* implementation for Array */;
//    ^^^ type prefix
const optMap = f => xs => /* function body for Option */;
//    ^^^ type prefix
```
This is a straightforward approach. We can declare functions as usual and combine them to type dictionaries in place, as soon as ad-hoc polymorphic functions require them.

Here is a more complex example with several overloaded functions involved, which unfortunately does not involve the functor type class. I will replace this with an implementation including a functorial constraint as soon as an appropriate example comes to my mind:

```javascript
// ad-hoc polymorphic function

const foldMap = ({fold, append, empty}) => f =>
//               ^^^^^^^^^^^^^^^^^^^^^ overloaded function constraints
  fold(comp2nd(append) (f)) (empty());

// array instances

const arrFold = f => acc => xs => {
  for (let i = 0; i < xs.length; i++)
    acc = f(acc) (xs[i], i);

  return acc;
};

// number instances

const add = x => y => x + y;

const addEmpty = () => 0;

// auxiliary functions

const comp2nd = f => g => x => y =>
  f(x) (g(y));
  
const sqr = x => x * x;
  
// MAIN

const xs = [1, 2, 3];

foldMap({fold: arrFold, append: add, empty: addEmpty})
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ type dictionary
  (sqr)
    (xs); // 14
```
[run code](https://repl.it/repls/HighlevelOblongDatamart)