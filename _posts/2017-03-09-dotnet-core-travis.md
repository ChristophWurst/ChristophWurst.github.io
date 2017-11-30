---
layout: single
title: Installing .NET Core 1.1.1 SDK on Travis CI
date: 2017-03-09
comments: true
tags: dotnet
---

Yesterday I wanted to enable Travis CI for an university project where we're using [.NET Core](https://www.microsoft.com/net/core). Although [Travis travis currently supports .NET Core builds](https://docs.travis-ci.com/user/languages/csharp/#Choosing-runtime-and-version-to-test-against), it does not yet include the new [.NET Core tools 1.0](https://blogs.msdn.microsoft.com/dotnet/2017/03/07/announcing-net-core-tools-1-0/). A new version of the Core tools is necessary if you try to build projects using the `.csproj` Project files instead of the `project.json` files.

It turned out to be fairly simple to install the sdk from Microsoft's Ubuntu repository, as documented [here](https://www.microsoft.com/net/core#linuxubuntu). All we had to do was adding the repository to Travis and installing the apt package:

```yaml
addons:
  apt:
    sources:
    - sourceline: 'deb [arch=amd64] https://apt-mo.trafficmanager.net/repos/dotnet-release/ trusty main'
      key_url: 'https://apt-mo.trafficmanager.net/keys/microsoft.asc'
    packages:
    - dotnet-dev-1.0.1
```

That's it. The Travis image was then able to install packages of all projects of the solution and build everything. I'm sure Travis will soon update their images and documentation to make this installation easier. You can find the full [.travis config file on GitHub](https://github.com/ChristophWurst/dotnet_core_travis/blob/2fd055b7dbbb74323618de95bbefd66540d4c55b/.travis.yml).

## References
* [Announcing .NET Core Tools 1.0](https://blogs.msdn.microsoft.com/dotnet/2017/03/07/announcing-net-core-tools-1-0/)
* [Travis' C# documentation](https://docs.travis-ci.com/user/languages/csharp/#Choosing-runtime-and-version-to-test-against)
* [Sample project on GitHub](https://github.com/ChristophWurst/dotnet_core_travis)
* [Travis successfully building the solution](https://travis-ci.org/ChristophWurst/dotnet_core_travis/builds/209280827)

