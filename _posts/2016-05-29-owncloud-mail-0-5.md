---
layout: post
title: ownCloud Mail 0.5 released
date: 2016-05-29
---

Following the [release early, release often philosophy](https://en.wikipedia.org/wiki/Release_early,_release_often) when working on ownCloud Mail, we do releases [quite often](https://github.com/owncloud/mail/releases). The most important enhancement of this release is our first step towards tight integration with more community apps. Additionally, it marks the beginning of [Tahaa's GSOC coding period](https://owncloud.org/blog/owncloud-and-google-summer-of-code/).

## ICS import

One of the most important reasons for creating another web-based mail app instead of simply using an existing one is the integration with other ownCloud apps.

So far, ownCloud Mail offered integration with contacts/CardDAV, which enabled the ability to show contact images for message senders and offers auto-completion of contact email addresses when writing new mails.

When writing a message, it's also possible to attach files selected from the ownCloud file system. Moreover, any received attachment can be saved to that file system. This eliminates the need to download (and upload) any file, it happens right in your cloud on any device.

#### The missing piece
Considering ownCloud Contacts and ownCloud Calendar two of the most important and commonly used community apps, there is a strong need to fill the gap and integrate with calendar too. There are many ideas on how we could integrate with the calendar app, but up to now there was no progress into that direction.

#### The current solution
Platforms like [meetup.com](https://meetup.com) attach .ics files of events when sending reminders. When using the mail app, you have to download the attachment either locally or to the ownCloud file system to open it then with either the calendar app or an external application. This workflow is inconvenient and unnecessary as it creates files that are only needed temporarily and therefore spam your file system.

### The idea
To make the integration as simple as possible from an user's perspective, the import should be possible in the mail app directly. Since ownCloud 9.0 the CalDAV back-end is integrated into core, hence we actually have no dependency on any another app.

From a design perspective it made sense to add another button next to the `Download` and `Save to ownCloud` button of attachments. The tricky part is that a user can have multiple calendars, so there must be a way to select the calendar used for the import. Luckily we have the well-known popover menus in ownCloud that can be used easily with some skills in CSS and JavaScript.

![]({{ site.url}}/assets/mail_ics_import_button.png)

Therefore the list of writable calendars is fetched from the CalDAV backend and listed in a simple popover on the import button. The user can then select the desired calendar, which starts the import process. If any error occurs, e.g. no writable calendar found or parsing error while reading the ICS data, the well-known yellow error notification is used to show a human-readable error message.

![]({{ site.url}}/assets/mail_ics_import_popover.png)

### The implementation
Like other DAV apps (e.g. files, calendar, contacts), {Evert}'s davclient is used for the DAV communication. When the import button is clicked, a list of the user's calendars is fetched using a PROPFIND request. Parsing that response was more work to do than expected, but luckily the calendar app already had JavaScript code that could be copied and adapted.

As soon as the user clicks on one of the listed calendars, the ICS content is downloaded from the server and parsed using Mozilla's ICAL library. This parsed content is then split into single components (e.g. events or todos) and sent to the CalDAV back-end using PUT requests.

### Future enhancements
Importing .ics attachments is just the first step for integrating with calendar. There are other great ideas we'd like to have, of which some are already tracked in the GitHub issue tracker:

* Show calendar attachment preview
* ITIP/iMIP Support
* Create tasks from incoming messages (a.k.a. don't use your inbox as todo list)
* Create and attach ICS files when sending a new message
* Detect dates in messages and show availability (useful for scheduling meetings)

## GSOC
This year ownCloud is one of the [accepted organizations for GSOC](https://summerofcode.withgoogle.com/organizations/5272257945403392/) and I am mentoring [Tahaa](https://github.com/tahaalibra) on "Add Mail Account Management & Alias Support & Signature Management to the OwnCloud" which is an important enhancement for the Mail app.

We will continue to do small releases as soon as an important new feature is implemented or critical bugs are fixed, so stay tuned for more awesome features in ownCloud Mail soon!
