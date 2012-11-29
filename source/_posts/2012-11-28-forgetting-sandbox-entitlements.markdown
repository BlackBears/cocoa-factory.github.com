---
layout: post
title: "Forgetting Sandbox entitlements"
date: 2012-11-28 06:19
comments: false
categories: macos
---
I was recently prepping a Mac OS app for Sandbox compliance but couldn't figure out why the following error: `NSPOSIXErrorDomain Code=1 "The operation couldn’t be completed. Operation not permitted"`

Finally it dawned on me that even simple outgoing network requests from the app, need to be specifically enabled thusly.

{% img http://i.imgur.com/6hz3q.png %}

I wish the error were more descriptive or that the `userInfo` dictionary of the `NSError` at least had a Sandbox violation description key.

Questions or comments about this post?  Contact the author of this post `@NSBum`.
