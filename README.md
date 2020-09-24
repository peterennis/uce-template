# <em>µ</em>ce-template

A tiny toolless library with tools included.

**[Live Demo](https://webreflection.github.io/uce-template/test/)**


### About

Inspired by [Vue 3 "_One Piece_"](https://github.com/vuejs/vue-next/releases/tag/v3.0.0), _uce-template_ provides a custom builtin `<template>` element to define components in a _Vue_ fashion.

```html
<template is="uce-template">

  <style scoped>
  span { color: green }
  </style>

  <the-green>
    The <span>{{thing}}</span> is green
  </the-green>

  <script type="module">
  export default {
    setup() {
      return {thing: 'world'}
    }
  }
  </script>

</template>
```

Add at any time this library in your page, and [see it bootstrapping](https://codepen.io/WebReflection/pen/xxVMgZx?editors=1000) all defined components.

```html
<!-- the defined component -->
<the-green></the-green>

<!-- the uce-template library -->
<script async src="//unpkg.com/uce-template"></script>
```


## Features

  * **SSR** compatibility out of the box: components definitions land *once* so no duplicated templates are needed in both layout and *JS*
  * a simple **CLI** that converts any html page or component into its minified version and, optionally, *Babel* transpilation
  * **Custom Elements** based, including builtin extends, so that *IE11*, *Safari*, or any other browser, will work right away
  * optionally **lazy** `<template lazy>` component, to resolve their definition only when live
  * optionally **shadow**ed `<custom-element shadow>` components, and optionally shadowed `<style shadow>` styles
  * a variety of pre-defined modules to import, including a virtual `@uce/reactive` module, to create reactive *UIs*
  * a runtime *ESM -> CommonJS* **module** system, where relative dependencies are [resolved (once) lazily](#the-lazy-js-environment), but any imported [module can be pre-defined](#the-module-js-environment) through the `resolve(name, module)` exported utility
  * everything pre-bundled within *10K* gzipped budget, or *9K* via brotli: one library for thousand portable use cases 🦄


### CLI

While it's suggested to install the *CLI* globally, due some not-super-light dependency, it's still an `npx` command away:

```sh
# check all options and usage
npx uce-template --help

# works with files
npx uce-template my-component.html

# works with stdin
cat my-component.html | uce-template
```

That's it, but of course we should be sure that produced layout still works as expected 👍


### <template>

Any template that extends `uce-template` *must* contain at least a custom element in it, either regular, or built-in extend:

```html
<!-- register regular-element -->
<template is="uce-template">
  <regular-element>
    regular
  </regular-element>
</template>

<!-- register builtin-element as div -->
<template is="uce-template">
  <div is="builtin-element">
    builtin
  </div>
</template>
```

Any template *might* contain a single `<script>` tag, and/or one or more `<style>` definitions.


### <custom-element>

Each "*component*" might define itself with, or without, its own static, or dynamic, content.

Such *content* will be used to render each custom element once "*mounted*" (live) and per each reactive state change.

All **dynamic parts** must be wrapped within `{{dynamic}}` curly brackets as shown here:

```html
<my-counter>
  <button onclick={{dec}}> - </button>
  <span>{{state.count}}</span>
  <button onclick={{inc}}> + </button>
</my-counter>
```

The `state`, `dec`, and `inc` references will be passed along through the script node, if any.

Regarding **ShadowDOM**, its polyfill is not included in this project but it's possible to define a component through its *shadow root* by adding a *shadow* attribute:

```html
<my-counter shadow>
  <!-- this content will be in the shadowRoot -->
  <button onclick={{dec}}> - </button>
  <span>{{state.count}}</span>
  <button onclick={{inc}}> + </button>
</my-counter>
```

The `shadow` attribute is `open` by default, but it can also be specified as `shadow=closed`.


### <style>

A component can have *one or more* styles in it, within a specific *scope*:

  * a generic `<style>` will apply its content globally, useful to address `my-counter + my-counter {...}` cases, as example
  * a `<style scoped>` will apply its content prefixed with the Custom Element name (i.e. `my-counter span, my-counter button {...}`)
  * a `<style shadow>` will apply its content on top of the *shadowRoot*, assuming the component is defined with a `shadow` attribute

There is nothing special to consider here, except that *global* styles might interfere with *IE11* if too obtrusive, as once again *IE11* doesn't understand the `<template>` element purpose and behavior.


### <script type="module">

A definition can contain only *one script tag* in it, and such *script* will be virtually handled like a *module*.

Since *IE11* is *not* compatible with `<template>` elements, if the `type` is not specified, *IE11* will try to evaluate all scripts on the page right-away.

Accordingly, the `type` attribute can really have any value, as it's completely irrelevant for this library, but such value must not be IE11 compatible, and `module` is just one value that *IE11* would ignore.

The script *might* contain a `default export`, or even a `module.exports = ...`, where such export *might* have a `setup(element) { ... }` method that returns what the *dynamic* parts of the component expect:

```html
<script type="module">
import reactive from '@uce/reactive';
export default {
  setup(element) {
    const state = reactive({ count: 0 });
    const inc = () => { state.count++ };
    const dec = () => { state.count-- };
    return {state, inc, dec};
  }
};
</script>
```

The `@uce/reactive` helper makes it possible to automatically update the view whenever one of its properties changes.

To know more about reactive changes, please [read this Medium post](https://medium.com/@WebReflection/reactive-state-for-data-dom-78332ddafd0e).


#### The module JS environment

The component script definition happens in a virtual `Function` sandbox, where all *imports* and *exports* are normalized as *CommonJS*, and a `require(module)` utility is provided.

Like it is for *CommonJS*, the `require` utility always returns the same module, once such module has been provided.

In the previous example, the `@uce/reactive` is virtually predefined in *uce-template*, but there are other modules too:

  * [augmentor](https://github.com/WebReflection/augmentor#readme) to create any *hooked* wizardry we like
  * [qsa-observer](https://github.com/WebReflection/qsa-observer#readme) to monitor nodes if needed
  * [reactive-props](https://github.com/WebReflection/reactive-props#readme) to create any reactive alchemy, even if this is provided already via `@uce/reactive`
  * [uce](https://github.com/WebReflection/uce#readme) to eventually import `html`, `svg`, or `render` utilities from [uhtml](https://github.com/WebReflection/uce#readme)

While `import {html} from 'uce'`, and *uce* in general, is very helpful to compose inner nodes of a defined component, every other module is there only because this library uses these modules to work, and it wouldn't make sense to not provide what's already included in *uce-template*.

However, it is possible to **define any module** using the `resolve(name, module)` utility:

```js
import {resolve} from 'uce-template';

import MyLibrary from 'MyLibrary';
resolve('my-library', MyLibrary);
```

Alternatively, it is also possible to define modules via the `uce-template` class itself:

```js
customElements.whenDefined('uce-template').then({resolve} => {
  resolve('my-utility', {any(){}, module: 'really'});
});
```

Please note that modules are unique so it is encouraged to use real module names to avoid clashing within third parts libraries.


#### The lazy JS environment

Modules can also be **loaded at runtime**, but only if *relative* or if the *CDN* supports Cross Origin Requests.

```js
// provided by uce-template
import reactive from '@uce/reactive';

import doubleRainbow from './js/rainbow.js';

export default {
  setup(element) {
    const state = reactive({ count: 0 });
    const inc = () => { state.count++ };
    const dec = () => { state.count-- };
    console.log(doubleRainbow); // 🌈🌈
    return {state, inc, dec};
  }
};
```

The `js/rainbow.js` in example should be reachable, and contain some export either via *ESM* syntax, or *CJS*, so that `export default '🌈🌈'` or `module.exports = '🌈🌈'` would be both valid and accepted.

It is *our duty* to be sure that lazily loaded modules can run within our target browsers, so in case *IE11* is one of these targets, it's our duty to transpile those file in a compatible way via *Babel* or others, as the `require` utility only cares about *imports* and *exports*.

The **advantage** of having *lazy* modules resolution is that a component defined via `<template is="uce-template" lazy>` will *not* need to have all dependencies pre-defined/resolved, and it *will* download once these only when any instance of such component is spot live.

As it is for ESM and CommonJS, every module is granted to be downloaded once and persist across multiple *imports*.
