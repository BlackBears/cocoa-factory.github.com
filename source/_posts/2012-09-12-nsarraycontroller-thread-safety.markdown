---
layout: post
title: "NSArrayController thread safety"
date: 2012-09-12 03:18
comments: true
categories: [cocoa,bindings,mac os, concurrency] 
---
`NSArrayController` is a work-horse of Cocoa bindings.  We recently ran into a problem with thread-safety which is not mentioned [in the documentation](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/ApplicationKit/Classes/NSArrayController_Class/Reference/Reference.html).

Consider this piece of code that downloads some resources and populates an `NSArrayController`:

``` objc
    - (void)downloadUsers {
        CCFUserListDownloader *downloader = [[CCFUserListDownloader alloc] initWithSessionID:self.sessionID];
        [downloader downloadWithCompletionBlock:^(CCFAPIStatus status,NSArray *remoteObjects) {
            [remoteObjects enumerateObjectsUsingBlock:^(id obj, NSUInteger idx, BOOL *stop) {
                CCFRemoteUser *user = [[CCFRemoteUser alloc] initWithDictionary:obj];
                if( user )
                    [[self userArrayController] addObject:user];
            }];
            [[self userArrayController] removeObjectAtArrangedObjectIndex:0];
    }];
```

Occasionally, we would get crashes on startup traced to this method.  The problem is that our `userArrayController` has multiple bindings to the UI.  Since `downloadUsers` was being executed on a background queue, its completion block was executed on the same queue.  When we add a `user` object to the array controller on a queue other than main, we would occasionally crash.  The solution is simple, just wrap the `addObject` call in an asynchronous dispatch to the main queue:

``` objc
    dispatch_async(dispatch_get_main_queue(), ^{
        [[self userArrayController] addObject:user];
    });
```
So, don't add objects to instances of `NSArrayController` on a background queue.