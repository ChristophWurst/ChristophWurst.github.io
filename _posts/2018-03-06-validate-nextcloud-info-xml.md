---
layout: single
title: Validate Nextcloud's info.xml on CI
comments: false
date: 2018-03-06
tags:
  - nextcloud
  - programming
  - foss
  - testing
header:
  image: /assets/20180306_validate-nextcloud-info-xml/banner.png
---

The Nextcloud [app metadata](https://docs.nextcloud.com/server/13/developer_manual/app/info.html)
is declared in an XML document, located at `appinfo/info.xml`. The structure of
this document is defined by a XML schema that can be downloaded
[here](https://apps.nextcloud.com/schema/apps/info.xsd).


Most code editors are capable of validating the `info.xml` while you're editing
it. To ensure that the document remains valid, it's possible to add a check
on the continuous integration build.


## xmllint

`xmllint` is a small CLI tool that can be used to validate `info.xml`. To do
that you first have to download the schema, e.g. with `wget` or `curl`.


Download, validation and cleanup are three simple steps:
```bash
wget https://apps.nextcloud.com/schema/apps/info.xsd
xmllint appinfo/info.xml --schema info.xsd --noout
rm info.xsd
```

## Travis CI

To run this check on [Travis CI], you have to make sure `xmllint` is available.
It can be installed by adding the `libxml2-utils` [apt package](https://docs.travis-ci.com/user/installing-dependencies/).


A working configuration can be found in the [Mail app](https://github.com/nextcloud/mail/blob/043de4b1af02081068bcd15e152f26a4df22ef52/.travis.yml#L83-L86).


[Travis CI]: https://travis-ci.org/
