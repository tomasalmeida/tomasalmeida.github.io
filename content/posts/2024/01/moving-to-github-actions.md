---
title: "Moving to GitHub Actions"
date: 2024-01-15T21:54:45+01:00
draft: false
tags: ["hugo", "travis-ci", "github", "actions"]
categories: ["development"]
ShowToc: true
TocOpen: true
weight: 2
---

## Why

Travis CI is blocking this blog to be generated, why? I do not know, I have credits for it, but it cannot be used, so I am forced to tick a task in my list and move to GitHub actions.

## Disabling Travis CI

### Removing GitHub <-> Travis CI integration

1. Go to https://github.com/settings/installations
2. Search "Travis CI" and click Configure
3. Scroll down and click in the "Uninstall" button
4. Go to https://github.com/settings/apps/authorizations
5. Search "Travis CI" and click Revoke

###  Deleting Travis CI

1. According to [this forum question](https://travis-ci.community/t/account-removal/13164), there is no UI option.
2. Send a mail to support@travis-ci.com requesting the account deletion.

## Add GitHub Actions

That is the easiest part as Hugo created a [nice tutorial](https://gohugo.io/hosting-and-deployment/hosting-on-github/). I followed it and that is the reason why you are able to read this blog post.
