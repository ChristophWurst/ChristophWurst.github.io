---
layout: single
title: A Modern HTTP Client for the Nextcloud Front-End
comments: true
#date: 2018-10-11
draft: true
tags:
  - foss
  - nextcloud
  - javascript
---

The Nextcloud front-end development is undergoing an important and overdue change where we
say goodbye to our friend jQuery and migrate our code bases to modern frameworks like Vue.

jQuery -- although being more a library than a framework -- is used extensively in many parts
of Nextcloud. It makes DOM operations easier, offers some UI helpers with *jQuery UI* and
lets us send browser-independent AJAX requests via `$.ajax`.

I'm not going into detail why we think Vue and it's friends are superior for our project, but
I want to briefly cover how HTTP communication, especially in regards to AJAX requests have
developed in the apps I maintain.

## The Traditional Way

You've probably seen code like this:

```js
$.get('/apps/mail/api/accounts', {
  success: function(accounts) {
    for (var i = 0; i < accounts.length; i++) {
      $.get('/apps/mail/api/accounts/' + accounts[i].id + '/folders, {
        ...
      });
    }
  },
  error: function(err) {
    console.error('could not load accounts, err);
  },
});
```

