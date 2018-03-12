---
layout: single
title: State of U2F in Firefox
comments: false
date: 2018-03-13
tags:
  - security
  - u2f
  - nextcloud
  - firefox
  - floss
header:
  image: /assets/20180313_state-of-firefox-u2f/header.png
---

Users of the [Nextcloud U2F app](https://apps.nextcloud.com/apps/twofactor_u2f) 
may have noticed that the app warns that only Chrome natively supports U2F and
an addon is necessary for Firefox:

> Install the "U2F Support Add-on" on Firefox to use U2F, this is not needed on Chrome.

However, if you search for that addon, you'll quickly notice that it's not
compatible with the current version of Firefox:

![U2F Support Add-on](/assets/20180313_state-of-firefox-u2f/extension.png)


Does that mean we cannot use U2F tokens on Firefox anymore? Yes and now. This
addon is not usable anymore but native U2F support has been added to Firefox
Nightly a few months ago:

<blockquote class="twitter-tweet" data-lang="de"><p lang="en" dir="ltr">There&#39;s experimental <a href="https://twitter.com/hashtag/U2F?src=hash&amp;ref_src=twsrc%5Etfw">#U2F</a> token support in <a href="https://twitter.com/firefox?ref_src=twsrc%5Etfw">@Firefox</a> 57 &amp; <a href="https://twitter.com/FirefoxNightly?ref_src=twsrc%5Etfw">@FirefoxNightly</a> behind the pref &quot;security.webauth.u2f&quot;; Thanks <a href="https://twitter.com/ttaubert?ref_src=twsrc%5Etfw">@ttaubert</a> &amp; <a href="https://twitter.com/qDot?ref_src=twsrc%5Etfw">@qDot</a>! <a href="https://t.co/CZH2Z9S7Ap">pic.twitter.com/CZH2Z9S7Ap</a></p>&mdash; J.C. Jones (@jamespugjones) <a href="https://twitter.com/jamespugjones/status/912314952232267777?ref_src=twsrc%5Etfw">25. September 2017</a></blockquote>

I tried it right away and it seemed to work with the Nextcloud U2F app.
You can use U2F on Firefox Quantum now, but you have to
[enable it in the settings](https://www.yubico.com/2017/11/how-to-navigate-fido-u2f-in-firefox-quantum/)
because it is disabled by default.


The good news is that Firefox will [soon support hardware tokens by default](https://hacks.mozilla.org/2018/01/using-hardware-token-based-2fa-with-the-webauthn-api/),
making U2F available for everyone. Once this is officially supported, we'll
[remove the Firefox warning](https://github.com/nextcloud/twofactor_u2f/issues/69)
from the [Nextcloud U2F app](https://apps.nextcloud.com/apps/twofactor_u2f).