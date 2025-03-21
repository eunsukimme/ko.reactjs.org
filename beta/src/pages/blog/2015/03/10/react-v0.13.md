---
title: 'React v0.13'
author: [sophiebits]
---

Today, we're happy to release React v0.13!

The most notable new feature is [support for ES6 classes](/blog/2015/01/27/react-v0.13.0-beta-1.html), which allows developers to have more flexibility when writing components. Our eventual goal is for ES6 classes to replace `React.createClass` completely, but until we have a replacement for current mixin use cases and support for class property initializers in the language, we don't plan to deprecate `React.createClass`.

At EmberConf and ng-conf last week, we were excited to see that Ember and Angular have been working on speed improvements and now both have performance comparable to React. We've always thought that performance isn't the most important reason to choose React, but we're still planning more optimizations to **make React even faster**.

Our planned optimizations require that ReactElement objects are immutable, which has always been a best practice when writing idiomatic React code. In this release, we've added runtime warnings that fire when props are changed or added between the time an element is created and when it's rendered. When migrating your code, you may want to use new `React.cloneElement` API (which is similar to `React.addons.cloneWithProps` but preserves `key` and `ref` and does not merge `style` or `className` automatically). For more information about our planned optimizations, see GitHub issues
[#3226](https://github.com/facebook/react/issues/3226),
[#3227](https://github.com/facebook/react/issues/3227),
[#3228](https://github.com/facebook/react/issues/3228).

The release is now available for download:

- **React**  
  Dev build with warnings: <https://fb.me/react-0.13.0.js>  
  Minified build for production: <https://fb.me/react-0.13.0.min.js>
- **React with Add-Ons**  
  Dev build with warnings: <https://fb.me/react-with-addons-0.13.0.js>  
  Minified build for production: <https://fb.me/react-with-addons-0.13.0.min.js>
- **In-Browser JSX transformer**  
  <https://fb.me/JSXTransformer-0.13.0.js>

We've also published version `0.13.0` of the `react` and `react-tools` packages on npm and the `react` package on bower.

---

## Changelog {/*changelog*/}

### React Core {/*react-core*/}

#### Breaking Changes {/*breaking-changes*/}

- Deprecated patterns that warned in 0.12 no longer work: most prominently, calling component classes without using JSX or React.createElement and using non-component functions with JSX or createElement
- Mutating `props` after an element is created is deprecated and will cause warnings in development mode; future versions of React will incorporate performance optimizations assuming that props aren't mutated
- Static methods (defined in `statics`) are no longer autobound to the component class
- `ref` resolution order has changed slightly such that a ref to a component is available immediately after its `componentDidMount` method is called; this change should be observable only if your component calls a parent component's callback within your `componentDidMount`, which is an anti-pattern and should be avoided regardless
- Calls to `setState` in life-cycle methods are now always batched and therefore asynchronous. Previously the first call on the first mount was synchronous.
- `setState` and `forceUpdate` on an unmounted component now warns instead of throwing. That avoids a possible race condition with Promises.
- Access to most internal properties has been completely removed, including `this._pendingState` and `this._rootNodeID`.

#### New Features {/*new-features*/}

- Support for using ES6 classes to build React components; see the [v0.13.0 beta 1 notes](/blog/2015/01/27/react-v0.13.0-beta-1.html) for details.
- Added new top-level API `React.findDOMNode(component)`, which should be used in place of `component.getDOMNode()`. The base class for ES6-based components will not have `getDOMNode`. This change will enable some more patterns moving forward.
- Added a new top-level API `React.cloneElement(el, props)` for making copies of React elements – see the [v0.13 RC2 notes](/blog/2015/03/03/react-v0.13-rc2.html#react.cloneelement) for more details.
- New `ref` style, allowing a callback to be used in place of a name: `<Photo ref={(c) => this._photo = c} />` allows you to reference the component with `this._photo` (as opposed to `ref="photo"` which gives `this.refs.photo`).
- `this.setState()` can now take a function as the first argument for transactional state updates, such as `this.setState((state, props) => ({count: state.count + 1}));` – this means that you no longer need to use `this._pendingState`, which is now gone.
- Support for iterators and immutable-js sequences as children.

#### Deprecations {/*deprecations*/}

- `ComponentClass.type` is deprecated. Just use `ComponentClass` (usually as `element.type === ComponentClass`).
- Some methods that are available on `createClass`-based components are removed or deprecated from ES6 classes (`getDOMNode`, `replaceState`, `isMounted`, `setProps`, `replaceProps`).

### React with Add-Ons {/*react-with-add-ons*/}

#### New Features {/*new-features-1*/}

- [`React.addons.createFragment` was added](/docs/create-fragment.html) for adding keys to entire sets of children.

#### Deprecations {/*deprecations-1*/}

- `React.addons.classSet` is now deprecated. This functionality can be replaced with several freely available modules. [classnames](https://www.npmjs.com/package/classnames) is one such module.
- Calls to `React.addons.cloneWithProps` can be migrated to use `React.cloneElement` instead – make sure to merge `style` and `className` manually if desired.

### React Tools {/*react-tools*/}

#### Breaking Changes {/*breaking-changes-1*/}

- When transforming ES6 syntax, `class` methods are no longer enumerable by default, which requires `Object.defineProperty`; if you support browsers such as IE8, you can pass `--target es3` to mirror the old behavior

#### New Features {/*new-features-2*/}

- `--target` option is available on the jsx command, allowing users to specify and ECMAScript version to target.
  - `es5` is the default.
  - `es3` restores the previous default behavior. An additional transform is added here to ensure the use of reserved words as properties is safe (eg `this.static` will become `this['static']` for IE8 compatibility).
- The transform for the call spread operator has also been enabled.

### JSX {/*jsx*/}

#### Breaking Changes {/*breaking-changes-2*/}

- A change was made to how some JSX was parsed, specifically around the use of `>` or `}` when inside an element. Previously it would be treated as a string but now it will be treated as a parse error. The [`jsx_orphaned_brackets_transformer`](https://www.npmjs.com/package/jsx_orphaned_brackets_transformer) package on npm can be used to find and fix potential issues in your JSX code.
