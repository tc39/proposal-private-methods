# Private methods and getter/setters for JavaScript classes

Daniel Ehrenberg

[Stage 4](https://tc39.es/process-document/)

Keeping state and behavior private to a class lets library authors present a clear, stable interface, while changing their code over time behind the scenes. The [class fields](https://github.com/tc39/proposal-class-fields) proposal provides private fields for classes and instances, and this proposal builds on that by adding private methods and accessors (getter/setters) to JavaScript. With this proposal, any class element can be private.

For discussion about semantic details, see [DETAILS.md](https://github.com/littledan/proposal-private-methods/blob/master/DETAILS.md). This document focuses more on the end user experience and intuition.

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

  get #x() { return this.#xValue; }
  set #x(value) {
    this.#xValue = value;
    window.requestAnimationFrame(this.#render.bind(this));
  }

  #clicked() {
    this.#x++;
  }

  constructor() {
    super();
    this.onclick = this.#clicked.bind(this);
  }

  connectedCallback() { this.#render(); }

  #render() {
    this.textContent = this.#x.toString();
  }
}
window.customElements.define('num-counter', Counter);
```

To make methods, getter/setters or fields private, just give them a name starting with `#`.

With all of its implementation kept internal to the class, this custom element can present an interface which is basically just like a built-in HTML element. Users of the custom element don't have the power to mess around with any of its internals.

Note that this proposal provides private fields and methods only as declared up-front in a field declaration; private fields cannot be created later, ad-hoc, through assigning to them, the way that normal properties can. You also can't declare private fields or methods in object literals; for example, if you implement your class based on object literals, or adding individual methods to the prototype, or using a class framework, you cannot use private methods, fields or accessors.

## Relationship to other proposals


Decorators aren't part of this proposal, but it's designed to allow decorators to work on top of it. See [unified class features](https://github.com/littledan/proposal-unified-class-features) for an explanation of how they would work together.

This proposal adds only private *instance* methods, omitting private `static` methods, which are under consideration in [the static class features proposal](https://github.com/tc39/proposal-static-class-features/).

## Status

### Consensus in TC39

This proposal reached [Stage 3](https://tc39.github.io/process-document/) in September 2017. Since that time, there has been extensive thought and lengthy discussion about various alternatives, including:
- [JS Classes 1.1](https://github.com/zenparsing/js-classes-1.1)
- [Reconsideration of "static private"](https://github.com/tc39/proposal-static-class-features)
- [Additional use of the `private` keyword](https://gist.github.com/rauschma/a4729faa65b30a6fda46a5799016458a)
- [Private Symbols](https://github.com/zenparsing/proposal-private-symbols)

In considering each proposal, TC39 delegates looked deeply into the motivation, JS developer feedback, and the implications on the future of the language design. In the end, this thought process and continued community engagement led to renewed consensus on the proposal in this repository. Based on that consensus, implementations are moving forward on this proposal.

### Implementations

Several full implementations exist:

- Babel by Tim McClure
  - Private methods [added in v7.2.0](https://babeljs.io/blog/2018/12/03/7.2.0)
  - Private accessors ([added in V7.3.0](https://babeljs.io/blog/2019/01/21/7.3.0#private-instance-accessors-getters-and-setters-9101-https-githubcom-babel-babel-pull-9101))
- [TypeScript](https://github.com/microsoft/TypeScript/pull/42458) - unflagged in nightlies, heading for TypeScript 4.3
- [V8](https://www.chromestatus.com/feature/5700509656678400) - unflagged in Chrome 84 / Node 14.6
- [JSC](https://webkit.org/blog/11577/release-notes-for-safari-technology-preview-122/) - unflagged in in Safari TP 122
- [SpiderMonkey](https://bugzilla.mozilla.org/show_bug.cgi?id=1662557) - flagged
- [Moddable XS](https://blog.moddable.com/blog/secureprivate/)
- [QuickJS](https://www.freelists.org/post/quickjs-devel/New-release,82)
- [esbuild](https://esbuild.github.io/content-types/#javascript)
- [Additional tooling support](https://github.com/tc39/proposal-private-methods/issues/32)

### Activity welcome in this repository

You are encouraged to file issues and PRs this repository to
- Ask questions about the proposal, how the syntax works, what the semantics mean, etc.
- Propose and discuss small syntactic or semantic tweaks, especially those motivated by experience implementing or using the proposal.
- Develop improved documentation, sample code, and other ways to introduce programmers at all levels to this feature.

If you have any additional ideas on how to improve JavaScript, see ecma262's [CONTRIBUTING.md](https://github.com/tc39/ecma262/blob/master/CONTRIBUTING.md) for how to get involved.
