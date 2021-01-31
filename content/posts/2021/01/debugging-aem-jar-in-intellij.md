---
title: "Debugging AEM: java code in IntelliJ"
date: 2021-01-31T19:38:11+01:00
draft: false
tags: ["IntelliJ", "tricks", "java"]
categories: ["debug"]
ShowToc: true
TocOpen: false
weight: 2
---

When you are working on Adobe AEM, you usually have to debug some sling or OSGi classes to understand what is happening "behind the scene". IntelliJ allows you to download source code and attach it to IntelliJ, but sometimes you cannot do it (you do not have internet access, your vpn to your nexus is down, etc...). 

This trick allows you to import the jar bundles into IntelliJ and debug them without downloading the sources!

# Find which AEM bundle has...

## The class you want to debug

You are debugging your code and then, it does a call to a Sling/OSGi code and you want to debug it. So, go to your server folder and type:

```shell
find crx-quickstart/launchpad/felix -name '*.jar' -exec grep -Hls <CLASS> {} \;
```

Replace `<CLASS>` for the class you want to find. The results will be something like:

```shell
find crx-quickstart/launchpad/felix -name '*.jar' -exec grep -Hls ImportantClass {} \;
crx-quickstart/launchpad/felix/bundle12/version0.0/bundle.jar
crx-quickstart/launchpad/felix/bundle145/version0.0/bundle.jar
crx-quickstart/launchpad/felix/bundle200/version0.0/bundle.jar
```

So, now you know the bundles 12, 145 and 200 have the class you want to debug. 

## The message you see in the logs

Sometimes, you see a message that you do not know which class sent it. So let's search for the bundle which has this message:

```shell
find crx-quickstart/launchpad/felix -name "*.jar" -exec zipgrep "<MESSAGE>" '{}' \;
```

Again, the result will be similar to:

```shell
find crx-quickstart/launchpad/felix -name "*.jar" -exec zipgrep "found element" '{}' \;
crx-quickstart/launchpad/felix/bundle11/version0.0/bundle.jar
```

So, now you know the bundle 11 has the class you want to debug.

# Add the bundle to IntelliJ

1. Go to IntelliJ, open your project.
2. Right click in the main module > Open Module Settings
3. Open Libraries
4. Click in the + button (New Project Library)
5. Add new library (button + in the bottom of the screen)
6. Open the bundle you found in the previous steps.
7. Save everything

# Search your class

Now you can use shift + shift to open the search window (check the "Include non-project items").

If you found any error or want to help me to improve this article, please [comment the pull request](https://github.com/tomasalmeida/tomasalmeida.github.io/pull/3)
