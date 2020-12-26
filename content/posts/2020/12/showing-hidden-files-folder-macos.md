---
title: "Showing hidden files and folder on Finder (MacOS)"
date: 2020-12-26T11:49:09+01:00
draft: false
tags: ["macOS", "tricks", "terminal"]
categories: ["macOS"]
ShowToc: true
TocOpen: true
weight: 2
---

One thing that I miss from Windows is the Windows Explorer (yes, I like it). Finder is not as good as Windows Explorer...

## Permanent solution

I like to see all my files, so to do it on MacOS , open a terminal and type

```shell
defaults write com.apple.Finder AppleShowAllFiles true
killall Finder
```

### Revert changes
This setting is persisted even after a restart, so if you want to go back to your previous status (hidden files are hidden), open a terminal again and type:

```shell
defaults write com.apple.Finder AppleShowAllFiles false
killall Finder
```

## Finder shortcode

1. Open Finder
2. Hit Cmd + Shift + Dot key (`.`)

### Revert Changes

Repeat the same shortcode

1. Open Finder
2. Hit Cmd + Shift + Dot key (`.`)
