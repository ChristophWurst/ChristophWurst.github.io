---
layout: single
title: Unlocking Nextcloud Accounts Locked by Two-Factor Authentication
comments: true
date: 2018-10-08
tags:
  - foss
  - nextcloud
  - two-factor auth
---

I'm happy to announce a new two-factor provider for Nextcloud: the *Admin Support Provider*. In contrast to the existing providers like [TOTP] and [U2F], this provider is not meant to be set up by the users themselves. Instead, this one is used when users have lost access to their other factors and therefore the account is locked, making logins impossible. When the affected users contact the admins they can now trigger the generation of a temporary one-time code. With this code users can log in once to set up their other two-factor providers.

![](/assets/20181008_unlock_nextcloud_2fa/term1.png)

While this app is compatible with Nextcloud 14, it becomes more important with Nextcloud 15 where it is not possible to disable two-factor authentication
for single users anymore if they have set it up themselves.

## Documentation

You can find admin and user documentation on [Read the docs](https://nextcloud-twofactor-admin.readthedocs.io/en/latest/).

## Summary

Admins now have an easy way of helping their users gain access to an account that is otherwise locked by two-factor authentication. Feedback, bugs and questions may be submitted to [the issue tracker on GitHub](https://github.com/ChristophWurst/twofactor_admin/issues). If you like the app we also appreciate if you give it a star on GitHub :v: and rate it on the [Nextcloud App Store](https://apps.nextcloud.com/apps/twofactor_admin).

[TOTP]: https://github.com/nextcloud/twofactor_totp
[U2F]: https://github.com/nextcloud/twofactor_u2f
