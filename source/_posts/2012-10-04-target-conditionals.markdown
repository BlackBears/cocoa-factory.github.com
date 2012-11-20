---
layout: post
title: "Target conditionals"
date: 2012-10-04 14:04
comments: false
categories: mac os, ios
---
If you hang out on Stack Overflow long enough, you're bound to see a few questions about target conditionals like [this one - Preprocessor macro for OS X targets](http://stackoverflow.com/questions/12132933/preprocessor-macro-for-os-x-targets); so here's an attempt to sort it out:

##How do you conditionally compile for Mac and iOS platforms?##

This one is a little confusing because `TARGET_MAC_OS` doesn't really do what you think.

  Conditional                  | Mac OS | iOS device | iOS sim
  ---------------------------- | ------ | ---------- | ------- 
  `TARGET_OS_MAC`              |   X    |    X       |    X    
  `TARGET_OS_IPHONE`           |        |    X       |    X    
  `TARGET_OS_EMBEDDED`         |        |    X       |         
  `TARGET_OS_IPHONE_SIMULATOR` |        |            |    X    


(Credits to Greg Parker, [_Hamster Emporium Target Conditionals_](http://www.sealiesoftware.com/blog/archive/2010/8/16/TargetConditionalsh.html) for the table)

If you have one block of code for Mac and another for iOS, the trick is to place the iOS condition first:

``` objc
#if TARGET_OS_IPHONE
NSString *dataString = [NSString stringWithFormat:"data = %d",count];
#else
// this compiles on Mac OS
NSString *dataString = [NSString stringWithFormat:"data = %ld",count];
#endif
```

##How I support multiple iOS or Mac versions?##

Usually this means that the developer wants to incorporate features in a new OS version while maintaing backwards compatibility.  The solution has a few steps:

- Set the base SDK on the target build setting to the newest vesion that you need.
- Set the deployment target to the oldest version that you support

Having set the target build settings, we have to do some work in the code.  Let's say we want to incorporate iOS 5 features in an app that should support earlier versions, say iOS 4.3.  Then we'll set the base SDK to iOS 5+ and the deployment target to iOS 4.3.  In the code, we do something like this:

``` objc
#if __IPHONE_OS_VERSION_MAX_ALLOWED >= 50000
// compile this when the target base SDK is iOS 5.0 or greater.  
// We use this as a compile-time test to prevent errors if we 
// try to recompile the project with a lower base SDK setting
#endif
```
So far, we will be able to compile for platforms between the OS ranges specified by our lower boundary of the deployment target and our upper boundary of the base SDK.  But what happens at runtime?  We'll crash on any features above our deployment target; so we need a runtime check for class availability.  As of the iOS 4.2 SDK, you should use the `NSObject` `class` method to check for the availability of weakly linked classes at runtime.  This takes advantage of the `NS_CLASS_AVAILABLE` macro; so in our example:

``` objc
#if __IPHONE_OS_VERSION_MAX_ALLOWED >= 50000

if( [CIImage class]  ) {
    // CIImage is available in iOS 5.0
}

#endif
```

The first check is a compile-time check to ensure that we can compile on lower SDK's and the second check is a runtime check to ensure that the weakly linked `CIImage` class is available on the device where the app is running.  Of course, we can also test for method availability instead:

``` objc
#if __IPHONE_OS_VERSION_MAX_ALLOWED >= 50000

if( [UIImage respondsToSelector:@selector(someNewMethod)]  ) {
    // we can use fancy new method on UIImage
}

#endif
```

##What is `__IPHONE_OS_VERSION_MIN_REQUIRED?`##

Unless you set it yourself, the compiler will set it to your target deployment version.  Same for `__MAC_OS_VERSION_MIN_REQUIRED`.  But you probably guessed that already.