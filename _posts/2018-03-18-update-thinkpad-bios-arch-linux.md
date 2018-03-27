---
layout: single
title: Updating a Thinkpad BIOS on Arch Linux
comments: false
date: 2018-03-18
tags:
  - foss
  - howto
  - security
---

Lenovo released BIOS updates to fix security issues for CVE-2017-5715 (Spectre),
so I figures that it would be a good time to update mine. I've once bricked a
mainboard during a BIOS update, hence I have been quite hesitant when it comes
to updating BIOSes, but this time I didn't want to procrastinate.


BIOS updates are provided in two ways: executables for Windows and ISO images.
Obviously, I cannot use the Windows executables on Arch Linux, so I downloaded
the ISO images and naively though I could just `dd` them on a spare pen drive.
However, this drive is not bootable. I've found there seem to be multiple ways
to create a bootable drive from the ISO, but most of them seemed a bit too
complicated for this rather simple task.


The `geteltorito` tool extracts a bootable image from Lenovo's ISO images and
is available as [AUR package](https://aur.archlinux.org/packages/geteltorito/).


The process of creating a bootable image and flashing it onto the pen drive is
simple:

```bash
teltorito.pl -o bios.img gkuj17ww.iso
sudo dd if=bios.img of=/dev/sdb
sync
```


The update went smooth. The only weird thing was that the fan started spinning
on max RPM during the update process.

---

<small>
Update 2018-03-27: Replaced externally hosted image with embedded one.
</small>