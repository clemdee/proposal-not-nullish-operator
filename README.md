# Not-nullish `?` Operator

Champion: looking for one :)

Author: [@ClemDee](https://github.com/ClemDee)

Status: 0 (strawperson)

[Original TC39 thread](https://es.discourse.group/t/nullish-unary-operator/657)


## Goal

The goal of this proposal is to simplify a very a common use case when handling _nullish_ values, which is **evaluating whether a variable is nullish or not**. This proposal aims to be in continuity with previous proposals `??`, `??=` and `?.`.

The idea is to add the new `?` unary operator:

```js
if (?variableA) {
  // do something if a is defined (i.e. not undefined or null)
}
```


## Motivation

Over the past years, several proposals have been made to add convenience for handling nullish values:
- the **optional chaining** operator `?.`
- the **nullish coalescing** operator `??`
- the **logical nullish assignment** `??=`

However, a use case is still missing, and yet, a very simple and common one: checking whether a variable is _nullish_ or not.

For now, we would still have to write:

```js
if (variableA !== undefined && variableA !== null) {
 // ...
}
```

Or using the loose equality:

```js
if (variableA != null) {
  // ...
}
```

However, using loose equality is considered as a bad practise, as it is one of the most awkward feature of javascript. Even though linters can be configured to only allow loose equality when comparing with _null_ (and is a commonly used exception to this syntax), this feels "hack-y", for lack of better.

Adding a dedicated syntax for this use case would also make the language less confusing for new-commers, and could hopefully lead to banning loose equality for good.


## Proposal

So the goal of the proposal is to add the **not-nullish** `?` unary operator:

```js
if (?variableA) {
  // do something if variableA is defined
}
```

This would desugar to something like:

```js
function isDefined (variable) {
  return variable !== undefined && variable !== null;
}

if (isDefined(variableA)) {
  // do something if variableA is defined
  // note that variableA was only evaluated once
}
```

This new operator would improve both writability and readability compared to today available solutions.


### Example use with other operators

This operator combines really well with other operators of the langage:

- We can easily evaluate if a variable is _nullish_, combining this operator with the **logical not** `!` operator:

```js
if (!?variableA) {
  // do something if variableA is nullish
}
```

- This operator also goes really well along with the **optional chainning** `?.` operator:

```js
if (?myObj?.sub?.value) {
  // do something if myObj, sub and value are not nullish
}
```


### Operator precedence

The [operator precedence](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Operator_Precedence#Table) should be at the same level than the **logical not** operator (level 17), with _left-to-right_ associativity.

Also, using the `?` operator mutiple times in a row should be done using parentheses, else it will throw a syntax error:

```js
if (?(?variableA)) { /* ... */ }
```

### Handling undeclared variables

The `?` operator should not handle undeclared variables (like when using the `typeof` syntax: `typeof a === "undefined"`).
Other nullish operators `??` and `?.` throw a _ReferenceError_ exception if the variable is not declared, so it should stay consistent.

### Backward compatibility

The proposed `?` operator should not break any previous code.

While a `?` operator is already used for ternary condition `condition ? ifTrue : ifFalse`, as its syntax expects 3 members and a second `:` operator, there is not risk that it would be mistaken by Javascript engines / parsers.


## Q&A:

### Why not using a postfix operator, like the [existential operator](https://coffeescript.org/#existential-operator) in Coffeescript?

Most operators in Javascript are prefix, and having this operator being suffix would lead to awkward syntaxes.  
E.g. when checking if a variable is _nullish_:

```js
if (!variableA?) { /* ... */ }
```

compared to the much cleaner prefix version:

```js
if (!?variableA) { /* ... */ }
```

Also, the suffixe version could introduce [breaking changes](https://es.discourse.group/t/nullish-unary-operator/657/5).


### [Yet another way](https://stackoverflow.com/questions/3390396/how-can-i-check-for-undefined-in-javascript) of checking undefined variables?

Indeed, there are already too many ways of checking for undefined variables. This proposal aims to become the preferred way to handle both _undefined_ and _nullish_ values, in a ES6+ spirit.  
This could replace almost every existing ways of checking _undefined_: 

- `variableA !== undefined && variableA !== null`
- `variableA != null`
- `variableA !== undefined`
- `variableA !== void 0`

However, this would not replace:
- `typeof undeclaredVariable === "undefined"`

