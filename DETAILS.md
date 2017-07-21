# Semantic details for decorators

## Method syntax, not scoped functions for now

TC39 has considered adding function declarations which are inside of a class body and lexically scoped to be accessible
only inside of it. In this proposal, method-style declarations are instead pursued in order to provide a better developer
experience. The intuition that this drives towards is that private methods are just like public ones, just accessible
exclusively from inside the class.

### Private method syntax

```js
class Counter extends HTMLElement {
  #x = 0;


  connectedCallback() { #render(); }

  #render() {
    this.textContent = #x.toString();
  }
}
```

- Similar to other methods
- Easy to change public <-> private

### Alternative: Lexically scoped functions

```js
class Counter extends HTMLElement {
  #x = 0;

  connectedCallback() { render.call(this) }

  function render() {
    this.textContent = #x.toString();
  }
}
```
- Not analogous to public methods
- The receiver needs to be passed explicitly with `call`.

## Type-checking the receiver

Private methods, as specified in this repository, are treated as own, immutable private fields, rather than lexically
scoped values. The difference in semantics comes up in cases like the following:

```js
class C {
  #foo() { alert("hi"); }

  bar() {
    #foo();
  }
}


C.prototype.bar.call();
```

If treated as a lexically scoped value, this code would successfully `alert("hi")`. However, if it's treated like a
private field, it would throw a TypeError. The reasoning is discussed [in this bug thread](https://github.com/littledan/proposal-private-methods/issues/1).


## Private accessors

This proposal provides private accessors, as you can see in the following code sample:

```js
class Counter extends HTMLElement {
  #xValue = 0;

  get #x() { return #xValue; }
  set #x(value) {
    #xValue = value; 
  }
}
```

At least some form of private methods is of pretty core importance so that behavior which manipulates private fields can
be effectively factored, and by contrast, private accessors are less important. They are introduced here with the expectation
that they may be useful in large classes, or in conjuncton with decorators. They also provide a sort of consistency: all
class elements are present in public and private forms.

If you have thoughts on private accessors, please comment on [their open bug](https://github.com/littledan/proposal-private-methods/issues/7)
to discuss the motivation.
