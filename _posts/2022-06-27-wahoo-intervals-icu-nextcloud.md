---
layout: single
title: "Uploading Wahoo workouts to intervals.icu"
date: 2022-06-27
tags:
  - foss
  - sports
  - intervals.icu
  - nextcloud
---

The Wahoo Elemnt companion app pulls workout files off my bike computer in the [.fit file format](https://developer.garmin.com/fit/protocol/). The workout can then be viewed, uploaded to a number of serivces and shared with other apps.

Unfortunately, [intervals.icu](https://intervals.icu/) is not on the list of integrations. This makes it impossible to directly, or even automatically, upload workouts there. Bummer.

## The fully manual route

If I want to I can attach the bike computer to my notebook via USB and upload the file to intervals from there. This is possible, but tedious.

## Taking a detour

It wasn't quite obvious to me at the beginning but it's possible to use the workout *sharing* feature to send the .fit file to another app. Sharing it to the Nextcloud Android app allows me to pick a destination folder and upload the workout file right into my cloud files.

If I then have the folder sync'ed to my notebook I can upload it to intervals more conveniently, without any cable. Still, this is a repetitive and scriptable task.

## The shortcut

Fortunately [intervals.icu](https://intervals.icu/) offers [a public API](https://forum.intervals.icu/t/api-access-to-intervals-icu/609) and a resource to push workouts to.

This API lends itself perfectly for an automation based on [Nextcloud Workflows](https://nextcloud.com/workflow/). All it takes is registering a new operation and listen for the *file post create* event.

As a user I can then set up a new workflow rule that takes .fit files and passes it to the intervals.icu operation that takes care of the upload.

![Setting up Nextcloud Workflows](/assets/20220627_wahoo_intervals_icu_nextcloud/user_setting.png)

The only thing left is to configure the operation with my intervals athlete ID and API key. I can add it to the `oc_preferences` table manually.

With this tiny workflow app my workouts can be sent to Nextcloud from my phone and show on intervals in a matter of seconds.

## Conclusion

I was able to automate a small but repetitive task in a matter of a few hours. The app was actually written seven months ago and I only had to update it to bump the info.xml for new Nextcloud releases.

By intervals.icu's popularity I expect it to be supported directly by Wahoo at some point. Until then I have my custom solution. As a side effect I'm also backing up my workouts to Nextcloud.

The app is open source and [can be found on Github](https://github.com/ChristophWurst/workflow_intervalsicu).
