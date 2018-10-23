---
layout: single
title: Mixing Nextcloud Globals into Vue
comments: true
date: 2018-10-22
last_modified_at: 2018-10-23
tags:
  - nextcloud
  - vue
  - javascript
  - l10n
  - foss
header:
  image: /assets/20181023_mixing_nextcloud_globals_into_vue/console_error_big.png
---

Nextcloud apps use global functions to access server APIs like the `t` function to translate
a given string. Within a Vue template, you can use this function like this:

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

So far the easiest and cleanest way to fix this I have found is writing a tiny [Vue mixin](https://vuejs.org/v2/guide/mixins.html):

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

## Testing

This trick is also handy for testing scenarios where the globals aren't available. There
a slightly adjusted mixin can provide stubs for Nextcloud's global functions:

```js
// src/tests/mixins/Nextcloud.js

export default {
  methods: {
    t: (app, str) => str
  }
}
```

---

Update 2018-10-23: fixed typo.