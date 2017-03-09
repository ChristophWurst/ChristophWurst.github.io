---
layout: post
title: Secure your Nextcloud with TOTP two-factor auth
date: 2016-09-08
tags: nextcloud
---

Starting with [Nextcloud 10](https://nextcloud.com/blog/secure-monitor-and-control-your-data-with-nextcloud-10-get-it-now/), it is possible to [plug custom two-factor auth apps into Nextcloud's authentication system](https://statuscode.ch/2016/06/nextcloud-two-factor-bruteforce-and-more/). One of the most popular second factors is *TOTP*, also known as *Google Authenticator*, for which I have created a [2FA provider app for Nextcloud](https://github.com/ChristophWurst/twofactor_totp).

## TOTP – how it works

> TOTP is an example of a hash-based message authentication code (HMAC). It combines a secret key with the current timestamp using a cryptographic hash function to generate a one-time password. The timestamp typically increases in 30-second intervals, so passwords generated close together in time from the same secret key will be equal.
>
> -- <cite>https://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm</cite>

That means your server and your smartphone app share a secret. Whenever a user wants to authenticate on Nextcloud, the smartphone app creates a new time-based password using the shared secret. After the users entered that password into Nextcloud, the server can use the shared secret to verify the password.

## Installing and enabling the app
You can grab the latest version from the app store, from within your Nextcloud's admin web interface. Alternatively, you'll find a packaged version of the app on the [project's GitHub page](https://github.com/ChristophWurst/twofactor_totp/releases).

![Go to the admin's app page](/assets/nc_totp_install_1.png)
![Find the app in the *Tools* category](/assets/nc_totp_install_2.png)

## Enabling TOTP on your account
Now log into a user's account and go to the personal settings page.

![Go to your personal settings](/assets/nc_totp_enable_1.png)

You'll see that the TOTP second factor is still disabled for your account. If you haven't installed a TOTP app yet, this is the step where you'll need one. Personally, I can recommend *OTP Authenticator*, which is available [on F-Droid](https://f-droid.org/repository/browse/?fdfilter=otp&fdid=net.bierbaumer.otp_authenticator). There are many other apps you can use, like the proprietary *Google Authenticator* for [Android](https://play.google.com/store/apps/details?id=com.google.android.apps.authenticator2) and [iOS](https://itunes.apple.com/us/app/google-authenticator/id388497605).
![TOTP is disabled by default](/assets/nc_totp_enable_2.png)

Now, click "Enable TOTP" to generate a new secret. You'll see the secret and a QR-code, that can be scanned by your TOTP app:
![The newly generated TOTP secret is shown. Next to it you'll see a QR-code](/assets/nc_totp_enable_3.png)

Depending on which TOTP smartphone app you're using, type in your generated secret or scan the QR code. And that's it. Now your account is protected by a second factor.

## Logging in using a TOTP code

Now let's log out and in again to see the 2FA challenge:

![The 2FA challenge page](/assets/nc_totp_login_1.png)

Select the TOTP provider. Next, you'll be prompted to enter a TOTP code. So grab your phone again, open the TOTP app and enter the 6-digit code that has been generated for you:

![Type in your TOTP code](/assets/nc_totp_login_2.png)

If everything was set up correctly, you'll be redirected to your Nextcloud account. Since the code is *time-based*, it's important that your server's and your smartphone's clock are *almost* in sync. A time drift of a few seconds won't be a problem.

![](/assets/nc_totp_enable_2.png)

## Enhancements planned for Nextcloud 11

Based on the feedback I already got, I've made some small enhancements that will be included in Nextcloud 11:

* If there's only one active 2FA provider for your account, the provider selection page can be skipped. See [github.com/nextcloud/server/pull/1169](https://github.com/nextcloud/server/pull/1169)
* To prevent an accidential lock out of your account in case you lose access to your second factor (e.g. you lose your phone), you can now generate and use backup codes. See [github.com/nextcloud/server/pull/1171](https://github.com/nextcloud/server/pull/1171)

## Known problems

* On some setups, the 2FA pages refresh every 5 seconds. This bug has been [fixed](https://github.com/nextcloud/server/pull/984) and [backported to Nextcloud 10](https://github.com/nextcloud/server/pull/1104), which will be released [as 10.0.1 in mid-September](https://github.com/nextcloud/server/wiki/Maintenance-and-Release-Schedule).

Please let me know about any other problems [on GitHub](https://github.com/ChristophWurst/twofactor_totp).

