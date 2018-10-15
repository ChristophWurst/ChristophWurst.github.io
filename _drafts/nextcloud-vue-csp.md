---
layout: single
title: Working with Vue and Nextcloud's strict CSP
comments: true
#date: 2018-10-11
draft: true
tags:
  - nextcloud
  - vue
  - javascript
  - foss
---

Nextcloud 15 applies a [stricter default *content security policy*](https://github.com/nextcloud/server/pull/11028)
 (CSP) which requires special treatment for JavaScript frameworks and libraries that use
*unsafe-eval*. In this post I'd like to give a quick overview of what developers
might have to change in their apps in order to use Vue for their apps.

## Webpack Devtool

Some [Webpack Devtools](https://webpack.js.org/configuration/devtool/) generate
source mappings via the `eval` function. Therefore an option must be chosen that
works without it. The [Webpack Documentation](https://webpack.js.org/configuration/devtool/)
has a nice comparison table of available devtools.

## Dynamic Webpack Module Loading

In order to load scripts dynamically, their `<script>` tags must have a
[`nonce` attribute](https://www.w3.org/TR/CSP2/#script-src-the-nonce-attribute). When
Webpack loads modules dynamically, it can add that attribute for you if you specify

```js
__webpack_nonce__ = btoa(OC.requestToken)
```

in your entry script (e.g. `main.js`).

## Vue Compiler vs. Vue Runtime

Vue templates can be compiled in the browser or pre-compiled in a build step.
In order to comply the strict CSP, you need to pre-compile all templates into
template functions in your build step. This will happen automatically if you
use Webpack and the `vue-loader` plugin.


Don't fall into the trap of instantiating Vue components like this if you want
to mount a single component to a DOM element:

```js
new Vue({
	el: '#mount-point',
	template: '<MyComponent/>',
	components: {
		MyComponent
	}
})
```

Since this snippet also uses a dynamic template, meaning the Vue render function
has to be evaluated in the browser, it will be blocked by the CSP.

I've found the following snippet to be a good replacement:

```js
const View = Vue.extend(MyComponent)
new View().$mount('#mount-point')
```

## Removing the Vue Compiler

As documented in the Vue docs, you don't need to ship the Vue compiler if
templates are pre-compiled anyway. Since pre-compiled render functions are
the only way to use Vue with the strict CSP, you can safely move to a
runtime-only environment and shave off a few KBs from your app's js bundle.

See the [Vue docs](https://vuejs.org/v2/guide/installation.html#Runtime-Compiler-vs-Runtime-only)
for more info.


## Conclusion

Vue and Nextcloud's strict CSP work well together if you slightly change
your configuration. Once applied, Vue builds work very well for both
development and production environments.
