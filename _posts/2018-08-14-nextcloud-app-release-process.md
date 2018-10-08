---
layout: single
title: Nextcloud App Release Process
comments: false
date: 2018-08-14
last_modified_at: 2018-10-08
tags:
  - foss
  - nextcloud
---

Over the course of the last week, I released a handful of Nextcloud app updates.
This process is similar for all my maintained apps, so I have developed a
workflow which I'd like to document here.

## Assumptions

This workflow assumes that

* You are about to release version `x.y.z`.
* Code is hosted on GitHub.
* GitHub milestones are used to assign issues and pull requests to (upcoming) releases.
* [Krankerl](/2017/11/28/krankerl-nextcloud-app-mgmt.html) is used for packaging the app.

## Release Preparation

* Make sure all changes (issues and pull request) are correctly assigned
  to the target release milestone `x.y.z`.

## Release Pull Request

* Make sure local `master` branch is up-to-date.
* Create `release/x.y.z` branch.
  + The version `x.y.z` is typically determined by the type of change, following [semver](https://semver.org/).
* Update version number.
  + Update `appinfo/info.xml`.
  + Update `package.json`¹.
  + Run `npm install`¹ (to update `package-lock.json`)
  + Commit the changes (e.g. with `Version bump`)
* Update the changelog
  + Tip: go through the milestone's PRs and issues
  + Tip: go through the commits with ```git log PREVIOUS_VERSION_TAG..HEAD --pretty=oneline  | grep -v tx-robot | grep -v "Merge " | grep -v "Bump "```
    * This filters out l10n updates, merges and [Dependabot](https://dependabot.com/) updates.
  + Commit the changes (e.g. with `Update changelog for x.y.z`).
* Push `release/x.y.z` branch to GitHub
* Open a new pull request
  * Assign this pull request to the `x.y.z` milestone and close it.
  * Assign/@mention reviewers and await their feedback.
  * Put pull request on hold if new bugs/regressions are found and
    pull in the changes after they have been integrated.
  * Merge after positive review.
* Update local `master` branch.
* Create git tag `vx.y.z`.
* Publish app archive.
  * Build package with `krankerl package`.
  * Push tag to GitHub.
  * Create new GitHub release.
  * Upload app tarball to the GitHub release.
* Publish app on [App Store](https://apps.nextcloud.com).
  * Copy asset URL from the GitHub release.
  * Run `krankerl publish URL`.

<small>
¹ Skip if `npm` is not in use.
</small>

## Follow-Up

* Create a milestone for the next release.

## Markdown Checklist

Here is a condensed checklist to be used as overview in the release pull
request's description.

```markdown
[Release checklist](https://blog.wuc.me/2018/08/14/nextcloud-app-release-process.html):
- [ ] Created release branch.
- [ ] Updated version number.
  - [ ] Updated `appinfo/info.xml`.
  - [ ] Updated `package.json` and ran `npm install`.
- [ ] Updated the changelog.
- [ ] Created release tag.
- [ ] Pushed tag to GitHub.
- [ ] Built package with `krankerl package`.
- [ ] Created new GitHub release.
- [ ] Published app on App Store.
```

## Conclusion

These steps are simple but I sometimes forget to do some of them (e.g. running
`npm install` after changing `package.json`).
This process has shown to work well with the apps I maintain. The workflow
fits well for both collaboratively developed apps as well as ones where I am
the only developer (without peer reviews).

Since this post serves as documentation for myself, I plan on updating the
details if any of the steps are changed.

---

Update 2018-08-24:
* Removed dangling footnote from markdown checklist
* Added reference to this post to markdown checklist

Update 2018-10-08:
* Simplified markdown checklist
