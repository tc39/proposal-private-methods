# Private methods and getter/setters for JavaScript classes

Daniel Ehrenberg

Keeping state and behavior private to a class lets library authors present a clear, stable interface, while changing their code over time behind the scenes. The [class fields](https://github.com/tc39/proposal-class-fields) proposal provides private fields for classes and instances, and this proposal builds on that by adding private methods and accessors (getter/setters) to JavaScript. With this proposal, any class element can be private.

Decorators aren't part of this proposal, but it's designed to allow decorators to work on top of it. See [unified class features](https://github.com/littledan/proposal-unified-class-features) for an explanation of how they would work together.

## A guiding example: Custom elements with classes

To define a counter widget which increments when clicked, you can define the following with ES2015:

```js
class Counter extends HTMLElement {
  get x() { return this.xValue; }
  set x(value) {
    this.xValue = value; 
    window.requestAnimationFrame(this.render.bind(this));
  }

  clicked() {
    this.x++;
  }

  constructor() {
    super();
    this.onclick = this.clicked.bind(this);
    this.xValue = 0;
  }

  connectedCallback() { this.render(); }

  render() {
    this.textContent = this.x.toString();
  }
}
window.customElements.define('num-counter', Counter);
```

## Field declarations

With the class fields proposal, the above example can be written as


```js
class Counter extends HTMLElement {
  xValue = 0;

  get x() { return this.xValue; }
  set x(value) {
    this.xValue = value; 
    window.requestAnimationFrame(this.render.bind(this));
  }

  clicked() {
    this.x++;
    window.requestAnimationFrame(this.render.bind(this));
  }

  constructor() {
    super();
    this.onclick = this.clicked.bind(this);
  }

  connectedCallback() { this.render(); }

  render() {
    this.textContent = this.x.toString();
  }
}
window.customElements.define('num-counter', Counter);
```

In the above example, you can see a field declared with the syntax `x = 0`. You can also declare a field without an initializer as `x`. By declaring fields up-front, class definitions become more self-documenting; instances go through fewer state transitions, as declared fields are always present.

## Private methods and fields

The above example has some implementation details exposed to the world that might be better kept internal. Using ESnext private fields and methods, the definition can be refined to:

```js
class Counter extends HTMLElement {
  #xValue = 0;

  get #x() { return #xValue; }
  set #x(value) {
    #xValue = value; 
    window.requestAnimationFrame(#render.bind(this));
  }

  #clicked() {
    this.x++;
    window.requestAnimationFrame(#render.bind(this));
  }

  constructor() {
    super();
    this.onclick = #clicked.bind(this);
  }

  connectedCallback() { #render(); }

  #render() {
    this.textContent = #x.toString();
  }
}
window.customElements.define('num-counter', Counter);
```

To make methods, getter/setters or fields private, just give them a name starting with `#`. A shorthand for `this.#x` is simply `#x`.

With all of its implementation kept internal to the class, this custom element can present an interface which is basically just like a built-in HTML element. Users of the custom element don't have the power to mess around with any of its internals.

Note that this proposal provides private fields and methods only as declared up-front in a field declaration; private fields cannot be created later, ad-hoc, through assigning to them, the way that normal properties can. You also can't declare private fields or methods in object literals; for example, if you implement your class based on object literals, or adding individual methods to the prototype, or using a class framework, you cannot use private methods, fields or accessors.
