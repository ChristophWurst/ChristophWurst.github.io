---
layout: post
title: Keep your Nextcloud app secure
date: 2016-11-29
comments: true
---

Nextcloud 11 will be released soon and this will be the first version of the Nextcloud server that uses the new app store at [apps.nextcloud.com](https://apps.nextcloud.com). All apps uploaded to this new app store are [signed](https://docs.nextcloud.com/server/11/developer_manual/app/code_signing.html) by the developers. Usually that means a few manual steps, but I've found a way to nicely automate this and I thought it was worth to share with other developers.

## The basics

Releasing an app requires the following steps:

1. Create a copy of the files you want to package
2. Combine those files into an archive
3. Sign the archive
4. Upload the archive to a download server
5. Register the archive on the app store

To sign your app, you first need a certificate. This is well documented in our [developer manual](https://docs.nextcloud.com/server/11/developer_manual/app/code_signing.html#how-to-get-your-app-signed). Next, you'll want to create a copy of your app's files where you only keep the ones you want to include in your release. Files you don't want to include are for example version control directories, tests and configuration files of various tools (travis, bower, npm, composer). This task can be easily achieved with **rsync**. It basically allows you to copy a source directory to another directoy, where you can specify files and file patterns that you want to exclude.

```bash
rsync -a \
	--exclude=.git \
	--exclude=build \
	--exclude=.gitignore \
	--exclude=.travis.yml \
	--exclude=.scrutinizer.yml \
        --exclude=CONTRIBUTING.md \
	--exclude=composer.json \
	--exclude=composer.lock \
	--exclude=composer.phar \
	--exclude=l10n/.tx \
	--exclude=l10n/no-php \
	--exclude=Makefile \
	--exclude=screenshots \
	--exclude=phpunit*xml \
	--exclude=tests \
	<source> <destination> 
```

Now, let´s package those files into an archive. That´s a piece of a cake:

```bash
tar -czf your_app.tar.gz your_app
```

Signing the app is a one-liner too:

```bash
openssl dgst -sha512 -sign you_app.key you_app.tar.gz | openssl base64
```

The openssl command will print the signature to your terminal. You´ll need this signature to register the app archive on the app store. Before that, the archive has to be uploaded to a publicly accessible server. If you´re hosting your app on GitHub, you can simply create a new release and upload the archive there.
Next, you can register new release at [apps.nextcloud.com](apps.nextcloud.com). Make sure you [register your app](https://nextcloudappstore.readthedocs.io/en/latest/developer.html#registering-an-app) before you register the first release. When registering the release you´ll need the URL to the archive and the signature. If all goes well, the app store will download your archive, check its metadata and save the release.

Note: you don´t have to register the app release manually, you can also use the [app store´s REST API](https://nextcloudappstore.readthedocs.io/en/latest/restapi.html#api-create-release) 🚀

So far, so good – but, to save you some time for the next releases of your awesome app, it´s a good idea to automate this process.

## Automate packaging and signing

Many Nextcloud apps use a Makefile to set things up. This is just perfect for this use case. Alternatively, you can also create a simple BASH script. A minimal Makefile to automate the steps above would look like this:

```Make
app_name=you_app
project_dir=$(CURDIR)/../$(app_name)
build_dir=$(CURDIR)/build/artifacts
sign_dir=$(build_dir)/sign
cert_dir=$(HOME)/.nextcloud/certificates

all: appstore

clean:
	rm -rf $(build_dir)

appstore: clean
	mkdir -p $(sign_dir)
	rsync -a \
	--exclude=.git \
	--exclude=build \
	--exclude=.gitignore \
	--exclude=.travis.yml \
	--exclude=.scrutinizer.yml \
        --exclude=CONTRIBUTING.md \
	--exclude=composer.json \
	--exclude=composer.lock \
	--exclude=composer.phar \
	--exclude=l10n/.tx \
	--exclude=l10n/no-php \
	--exclude=Makefile \
	--exclude=screenshots \
	--exclude=phpunit*xml \
	--exclude=tests \
	--exclude=vendor/bin \
	$(project_dir) $(sign_dir)
	tar -czf $(build_dir)/$(app_name).tar.gz \
		-C $(sign_dir) $(app_name)
	openssl dgst -sha512 -sign $(cert_dir)/$(app_name).key $(build_dir)/$(app_name).tar.gz | openssl base64
```

All you have to adjust in the Makefile above is your app´s name in the first line. The script assumes to find the certificate and private key in ``.nextcloud/certificates`` inside your home directory. This is the convention. Alternatively, you can of course adapt the command to your needs. The important part is to keep your private key out of your git repository, especially if you host it publicly.

Then you´re ready to package and sign your app with a single command:

```bash
make appstore
```

You'll find the archive in `build/artifacts/your_app.tar.gz`.

## Conclusion

As you can see, it´s not rocket science to create a packaged version of your Nextcloud app for [apps.nextcloud.com](https://apps.nextcloud.com). Still, it makes sense to automate the process in a small script to save yourself some time 
😉

## References
* [App store documentation for developers](https://nextcloudappstore.readthedocs.io/en/latest/developer.html)
* [Nextcloud developer manual](https://docs.nextcloud.com/server/11/developer_manual/app/code_signing.html)
* [Nextcloud 11 beta](https://nextcloud.com/blog/two-reasons-why-you-should-test-nextcloud-11-beta-this-weekend/)

