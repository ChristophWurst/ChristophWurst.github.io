---
layout: post
title: Adding two-factor auth to ownCloud
date: 2016-05-30
---

I have spent a couple of weeks working on a fundamental component of the ownCloud core, its authentication system. Starting with ownCloud 9.1 (which will be released in July 2016) it will be possible to plug custom two-factor auth providers into the ownCloud core for enhanced account security.

## Pluggable auth
Starting with refactoring and cleaning up of the existing auth system titled ["Pluggable](https://github.com/owncloud/core/issues/23458) [auth"](https://github.com/owncloud/core/pull/24189), I added token-based authentication to the ownCloud server. Token-based authentication finally makes it possible to connect client applications and devices without giving those your login password. Additionally, session tokens are created for browsers. This allows tracking and killing browser sessions [via the personal settings page of an ownCloud user](https://github.com/owncloud/core/pull/24703) remotely. This means if you lose your smart phone, you no longer have to change your password to disconnect all connected browsers and devices, you can now invalidate that specific device token with a single click.

![]({{ site.url}}/assets/oc_twofactor_5.png)

### Backward compatibility
The great news is that we are not breaking compatibility with old and third-party clients, as we still allow authentication via username and password. However, the preferred way to configure clients will now be to create a device token (a.k.a. device-specific password) and use that as password. The ownCloud core will detect that the password is actually a token and will use it to authenticate.

In case admins want to enforce token-based auth and disallow login via password (with the exception of the login page of course), we added a [config option](https://github.com/owncloud/core/pull/24811/files#diff-2f769bc01ffefdf14eb04404218e7582) that admins can set. In this case the ownCloud server will then deny authenticated requests if the password is the login password and not a device token.

## Two-factor auth
Token-based authentication is the foundation for second factor authentication. Based on [Lukas Reschke's](https://github.com/LukasReschke) [proof of concept pull request](https://github.com/owncloud/core/pull/19752) I added internal two-factor auth support to ownCloud. Specifically, core will take care of the communication and management of second factor providers that are plugged into the system as apps. By decoupling core interfaces and specific two-factor auth (2FA) providers we keep the ownCloud core easier to maintain while allowing admins to create and install 2FA providers they want without needing to modify the ownCloud core.

### Architecture overview
As mentioned above, developers can create their own specific second factor providers. After a successful login, ownCloud will check if there are apps providing 2FA support. If there is at least one, access to the account will be blocked until one of the 2FA challenges was solved. Hence, you won’t find any second factor if you just download and install a recent development version of ownCloud.

![]({{ site.url}}/assets/oc_twofactor_1.png)
![]({{ site.url}}/assets/oc_twofactor_2.png)
![]({{ site.url}}/assets/oc_twofactor_3.png)
![]({{ site.url}}/assets/oc_twofactor_4.png)

### Writing your own two-factor provider
Creating your own 2FA provider requires two important components. First, you have to create a class that implements the  [``OCP\Authentication\TwoFactorAuth\IProvider`` interface](https://github.com/owncloud/core/blob/master/lib/public/Authentication/TwoFactorAuth/IProvider.php). This class is the main communication component between your app and the ownCloud core. As a second step, you register the 2FA provider in the app’s ``info.xml`` in the ``two-factor-providers`` section:

```xml
<two-factor-providers>
	<provider>OCA\TwoFactor_SMS\Provider\TwoFactorSMSProvider</provider>
</two-factor-providers>
```

Additionally, it is necessary to tag the app as ``prelogin`` and ``authentication`` app, so it is loaded in an early stage:

```xml
<types>
	<prelogin/>
	<authentication/>
</types>
```

A very simple proof of concept provider was created as “TwoFactor Email”, which may be used as reference: [https://github.com/owncloud/twofactor_email](https://github.com/owncloud/twofactor_email). It registers a dead simple provider that is enabled for all users and whose challenge can be passed with the hard-coded password "passme".

### Disabling two-factor auth for specific users
As soon as a 2FA provider is found, users are required to solve the challenge before getting access to their account. However, it can happen that users lose access to their second factor (e.g. email verification is used and the mail server is down for some hours). For those eventualities, the admin can (temporarily) disable the second factor via an `occ` command:

```
$ ./occ twofactor:disable florian
Two-factor authentication disabled for user florian

$ ./occ twofactor:enable florian
Two-factor authentication enabled for user florian
```

### Implications of using two-factor auth
Please note that enabling a second factor will automatically enforce token-based authentication for clients. This means connected clients may be disconnected if they were configured with the login password.

## It's your turn
The goal of my changes for 9.1 was to have an enhanced and extensible architecture that allows app developers write their own second factor apps. I will implement the email provider soon, so we have at least one working example of an ownCloud two-factor provider. If you’re a developer and you’d like to write your own 2FA provider let me know and I’ll help with the implementation.
