---
layout: single
title: Mixing Nextcloud Globals into Vue
comments: true
#date: 2018-10-23
draft: true
tags:
  - nextcloud
  - vue
  - javascript
  - foss
---

Nextcloud apps use global functions to access server APIs like the `t` function to translate
a given string to the user's locale. Within a Vue template, you can use this function like
this:

```js
<template>
  {% raw %}<span>{{ t('myapp', 'Hello, {user}', {user: this.user})}}</span>{% endraw %}
</template>

<script>
  export default {
    props: ['user']
  }
</script>
```

However, this won't work. You will see a `TypeError: _vm.t is not a function` error in your
browser's console:

![TypeError: _vm.t is not a function](/assets/20181023_mixing_nextcloud_globals_into_vue/console_error_small.png)

So far the easiest and cleanest way of fixing this I have found is writing a tiny [Vue mixin](https://vuejs.org/v2/guide/mixins.html):

```js
// src/mixins/Nextcloud.js

export default {
  methods: {
    t
  }
}
```

This mixin makes the global function `t` available as instance method. It can be
used in an app globally via the entry script:

```js
// src/main.js

import Vue from 'vue'
import Nextcloud from './mixins/Nextcloud'

Vue.mixin(Nextcloud)
```

Alternatively, it can also be applied to specific components only:

```js
// src/components/MyComponent.js

<tempate>
  ...
</template>

<script>
  import Nextcloud from '../mixins/Nextcloud'

  export default {
    props: ['user'],
	mixins: [Nextcloud]
  }
</script>
```
