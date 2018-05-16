---
layout: single
title: Gaining Disk Space with `cargo clean`
comments: false
date: 2018-05-16
tags:
  - foss
  - rust
header:
  image: /assets/20180516_disk_space_cargo_clean/rust_is_not_a_crime_small.jpg
  image_description: "Rust is not a crime"
---

I just got the Gnome notification that my SSD once again runs out of free disk
space. This has happened before. Ususally I just clean up my pacman/pacaur cache with

```bash
pacaur -Sc
```

and run a few [Docker cleanup commands](https://www.calazan.com/docker-cleanup-commands/).


This time, this didn't give me much free space, as only ~5GB were freed. So I started
[`baobab`](https://wiki.gnome.org/action/show/Apps/DiskUsageAnalyzer) to analyze my disk.
To my surprise, a large portion of disk space is consumed by my `rust` projects directory.
The directory contains 55 subdirectories (projects), which sum up to ~50GB of disk space.


So I decided to run `cargo clean` on all these projects to get rid of large files I don't
need at the moment (I can recompile at any time, anyway).


```bash
for proj in $(find . -name Cargo.toml | xargs readlink -f | xargs dirname); do pushd $proj; cargo clean; popd; done
```

And voilà! The `rust` directory shrunk to 1.1GB, getting rid of 49GB unneeded storage.
