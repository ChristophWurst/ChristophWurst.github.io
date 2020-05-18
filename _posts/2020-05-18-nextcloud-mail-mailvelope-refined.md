---
layout: single
title: "Mailvelope in Nextcloud Mail, refined"
comments: false
date: 2020-05-18
last_modified_at: 2020-05-18
tags:
  - nextcloud
  - mail
  - announcement
header:
  image: /assets/20200518_nextcloud_mail_mailvelope_refined/encrypted_email.png
---

Nextcloud Mail has offered PGP encryption via a third-party browser extension [Mailvelope](https://mailvelope.com/en) for a while. Recently we discovered that there were some issues due to our rich text editor. This led me into revisiting the integration and my attempted *bugfix* turned into quite an *enhancement*.

## What we had before

Prior to Mail v1.3.5, our app was not even aware of the Mailvelope integration. The user configured their Nextcloud domain in the extension settings and Mailvelope would dected textareas and similar HTML inputs and show a button to open a popup with the encryption editor. The user then had to pick the recipients again (Mailvelope had no clue about the recipient selection in Nextcloud Mail) and once the message got encrypted, it was written back to the original window.

This solution worked okay-ish for a long time, but  never felt seamless due to the redundant recipient selection and the popup window.

## Switching to API mode

To get some help with fixing the integration after breaking with our CKEditor as rich text editor, I contacted Mailvelope support and got quick feedback from one of the developers. I was introduced into the API mode, where an application integrating Mailvelope could interact with the extension programmatically.

This means Nextcloud Mail is now capable of knowing whether Mailvelope is installed and enabled for the domain.

## The new sending experience

As Mail is now capable of detecting Mailvelope, it will show an encryption option in the new message composer.

![The "Ecrypt message with Mailvelope" option in Nextcloud Mail v1.3.5](/assets/20200518_nextcloud_mail_mailvelope_refined/actions.png)

Note that the *Send* button also now indicated whether whether the composed message will be sent encrypted or in plain text. Simple installations without Mailvelope will still only show *Send*, so the interface is less complex for those users.

Once the encryption option is enabled, the editor component will be replaced by an editor of Mailvelope.

![The Mailvelope editor in Nextcloud Mail v1.3.5](/assets/20200518_nextcloud_mail_mailvelope_refined/encrypted_email.png)

See, no more popup window! No more redundant recipient selection. Nextcloud Mail will pass the recipients to Mailvelope for you. In fact, it will even check if all the picked recipients have a key available and prevent sending if this condition is not met.

## Better reading experience

The switch to the *API version of Mailvelope* also allows us to embed a display container into Nextcloud Mail, so the experience feels a bit more integrated.

![The Mailvelope editor in Nextcloud Mail v1.3.5](/assets/20200518_nextcloud_mail_mailvelope_refined/display.png)

You can directly see and download the attachments.

## Smoother replies

When replying to an encrypted message, we will pass the raw data into Mailvelope. The extension will decrypt and quote the message for you, so it's very convenient to have encrypted conversations.

![The Mailvelope editor in Nextcloud Mail v1.3.5](/assets/20200518_nextcloud_mail_mailvelope_refined/reply.png)

## Get it now

Nextcloud ![Mail v1.3.5 has just been relased to the app store](https://apps.nextcloud.com/apps/mail). Upgrade and start encrypting more messages now :wink:

If you have any questions or feedback please let us know [on the forum](https://help.nextcloud.com/t/mailvelope-in-nextcloud-mail-refined/81967) :)
