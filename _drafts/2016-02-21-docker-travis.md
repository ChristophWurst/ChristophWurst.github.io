---
layout: post
title: Using Docker to speed up Travis builds
date: 2016-02-21
---
Depending on the complexity of continuous integration test matrices on Travis, running all test with various configurations can take some time. While that might not be an issue for most scenarious – e.g. testing the master branch of your project after a pull request has been merged – it's quite annoying to have to wait for test results for an hour if you depend on the those.

At [ownCloud Mail](https://github.com/owncloud/mail) we are testing pull requests agains different versions of the ownCloud Core, as well as all three supported database systems and all supported PHP versions. Naturall, we can't test all 48 combinations, hence we only test 8 typical combinations. Although this is a small number of build jobs, it still took abount four minutes to finish each job. That led to a total time of 35 to 45 minutes.

## Dependency caching
About a minute of each build job is spent on downloading and installing dependencies, so we enabled [dependency caching](https://docs.travis-ci.com/user/caching/) on [container-based Travis infrastructure](https://docs.travis-ci.com/user/workers/container-based-infrastructure) last year. This led to a noticable reduction of the total build time, see [this pull request](https://github.com/owncloud/mail/pull/1084) for more information on that.

## Using external servers
While most unit tests are run in an isolated environment with all dependencies mocked, some integrational tests need real servers they can speak to. OwnCloud Mail has a few test cases that talk to a real IMAP server and a Gmail test account was used for running those tests. This basically works, but leads to the following problems:

1. Different tests can not be run in parallel as they are using the very same IMAP server. That leads to dependencies between tests.
2. Aborted tests can make the following tests fail. Altough cleanup code is used to restore the original IMAP state so all tests are run with the same preconditions, inconsistent states may remain if tests are aborted.
3. Failing tests are not always reproducable by developers who don't have access to that Gmail account. Different IMAP servers, or even different account on the same IMAP server can raise different errors.

## Docker
To solve the problems above, a local, resetable solution had to be found. After reading that you can [use Docker in Travis builds](https://docs.travis-ci.com/user/docker/), I searched the [docker hub](https://hub.docker.com/) for a preconfigured Dovecot image and found an almost perfect one: [invokr/mail](https://hub.docker.com/r/invokr/mail/). The only missing feature was adding an IMAP account by script, therefore I forked the repository and created [my first automated docker build](https://github.com/ChristophWurst/owncloud-mail-test-docker).

Using that docker image for IMAP integration tests sped up build jobs a lot. Additionally, it made it possible for developers to run all tests locally.

This simple, but effective change fixed all the three problems from above: Travis build jobs can be run in parallel as each job has its own IMAP server and developers can use that docker image to test their changes locally before pushing.

## Further improvement

While reading how to use [Docker on Travis](https://docs.travis-ci.com/user/docker/) I read about other possible way for [speeding up builds](https://docs.travis-ci.com/user/speeding-up-the-build/):

### Disabling XDebug
PHPUnit uses XDebug to generate code coverage, which is then sent to [scrutinizer](https://scrutinizer-ci.com/). Unfortunately, having XDebug enabled while running PHPUnit unit tests can have a negative impact on performance. Therefore I changed the Travis bulid matrix to disable XDebug on all but one build. Given the fact that all build jobs create mostly the same code coverage and scrutinizer ignores all but the first report it receives, this has no disadvantage at all.

## Comparing the results
This is what a typical build job looked like before using a Docker image for IMAP testing:
![before using docker]({{ site.url }}/assets/travis_docker_before.png)

After adding Docker, enabling parallel builds and collecting code coverage on one build only I got the following result:
![after using docker]({{ site.url }}/assets/travis_docker_after.png)
