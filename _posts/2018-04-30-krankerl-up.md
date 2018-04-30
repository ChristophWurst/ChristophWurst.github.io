---
layout: single
title: Automated project setup with `krankerl up`
comments: false
date: 2018-04-30
tags:
  - foss
  - nextcloud
  - rust
header:
  image: /assets/20180430_krankerl_up/term.png
  image_description: "Krankerl up in action"
---

With the release of [Krankerl 0.7.0](https://github.com/ChristophWurst/krankerl/releases/tag/v0.7.0),
I've added a new `up` command that detects [composer](https://getcomposer.org/)
and [npm](https://www.npmjs.com/) availability and automatically installs all
Nextcloud app dependencies and runs the npm build task. For those who are not
familiar with [Krankerl](/2017/11/28/krankerl-nextcloud-app-mgmt.html): it's a
small CLI tool to help building Nextcloud apps, written in Rust.


The goal was to use my Krankerl tool for *any* Nextcloud apps, not just those
that I maintain and where a Krankerl config is available. By reading both
composer.json and package.json, Krankerl can safely determine whether these
package managers are used and if there is a build script defined for npm.


For interested Rustaceans: this feature was build with the help of two new
crates: `composer` and `npm_scripts`. You can find both of them on creates.io:
* [composer](https://crates.io/crates/composer)
* [npm_scripts](https://crates.io/crates/npm_scripts)
