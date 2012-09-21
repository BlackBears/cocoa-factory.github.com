---
layout: post
title: "What's a run loop anyway?  NSRunLoop 101"
date: 2012-09-06 18:10
comments: true
categories:  cocoa, networking, coreos
---
`NSRunLoop` is one of the mysterious classes in the frameworks that few seem to understand really well.  I recently encountered difficulties with some aynshcronous networking code (more on that in a future post) and thought I'd share what I learned about run loops.

## Facts about run loops ##

###Run loops manage input source events###

A run loop - `NSRunLoop` in Cocoa - is a class of objects that manages input sources, like user events (mouse, keyboard, etc.), `NSPort` events, and those that emanate from `NSConnection` objects.  You might think that the latter is the superclass of `NSURLConnection` objects; but you'd be wrong.  `NSURLConnection` is a subclass of `NSObject` - even though run loops manage events from `NSURLConnection` also.  So, you get the picture?  `NSRunLoop` manages events.

###Every thread gets a run loop###

If you create a thread, you get an `NSRunLoop` with it.

###(Most) run loops don't run by themselves###

You must explictly run any run loop other than the main thread run loop.

###Run loops that have no input sources don't run.###

Look closely at the documentation for `NSRunLoop` `run` method:  *"If no input sources or timers are attached to the run loop, this method exits immediately..."*  This means that if you want a run loop to keep turning, you need to find an event source to attach to it.  For example:

``` objc
NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
[runLoop addPort:[NSPort port] forMode:NSDefaultRunLoopMode];

// now when we start the runLoop, we have an event source
[runLoop run];
```

But simply removing all input sources from a run loop is not guaranteed to stop it.  IF you want to stop a run loop, you must explicitly do so:  `

``` objc
BOOL shouldRun = YES;	// this is in a "global" context
NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
while( shouldRun && [runLoop runMode:NSDefaultRunLoopMode beforeData:[NSDate distantFuture]] );

// set shouldRun to NO somewhere else to terminate the run loop
```

###Run loops have modes and you can create your own###

Run loops have modes that specify groups of input sources that are monitored for the run loop running in that mode.  Usually you will just use the `NSDefaultRunLoopMode`; but the others are:
	
`NSConnectionReplyMode` used with `NSConnection` objects
	
`NSModalPanelRunLoopMode` used with events associated with modal panels in OS X
	
`NSEventTrackingRunLoopMode` used with UI tracking events
	
`NSRunLoopCommonModes` is a configurable group of common modes.  In Foundation is includes all of the modes except `NSConnectionReplyMode` by default.  

If you want to create your own mode, just use a different string:

``` objc
NSRunLoop *runLoop = [NSRunLoop currentRunLoop];
[runLoop addPort:[NSPort port] forMode:@"com.cocoafactory.MySpecialMode"];
```
Apple recommends reverse DNS notation to avoid stepping on someone else's run loop.

###You rarely need to work directly with run loops###
Because the main run loop is vital to the application, the `run` method on `NSApplication` and `UIApplication` start the main run loop during the startup sequence. Otherwise, even for threads that you create yourself, you probably do not need to start its run loop.  If the thread needs to work with ports, input sources, timers, or certain connections, then you need to start and manage its run loop for those events.  More on some of those situations in a future post. 