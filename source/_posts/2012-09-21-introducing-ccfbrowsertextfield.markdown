---
layout: post
title: "Introducing CCFBrowserTextField"
date: 2012-09-21 06:10
comments: false
categories: [cocoa, mac os, goodies, open source]
---
We just released `CCFBrowserTextField` for public consumption.  Take a look at [our github repository](https://github.com/cocoa-factory/CCFBrowserTextField) to get started using it.  Here's what this `NSTextField` subclass looks like:

{% img right http://tinypic.com/r/2nlz5ah/6 %}

That's it.  A text field with an embedded document-like icon.  The intent is to provide a way of browsing to a path that should be contained in the text field without an external button.  It was inspired both a desire for a more compact, connected UI and by seeing someone else who had done something similar.  In the amazing [Omni Frameworks](https://github.com/omnigroup/OmniGroup) in OmniAppKit framework, you'll find a number of interesting widgets.  They created a text field with a calendar button that popups up a calendar picker.  We liked the idea and learned a lot about how text fields work, then adapted it to meet our needs.  By the way, Omni writes great software besides putting their frameworks out for public use.  I use at least one of their pieces of software every day.

###Using `CCFBrowserTextField`###

If you want to use `CCFBrowserTextField` just go to the [github repository](https://github.com/cocoa-factory/CCFBrowserTextField) and download it.  When you want to use it, just insert an `NSTextField` object into your interface and set its class type to `CCFBrowserTextField`.  For the button to be active, you just need to provide it with an action block:

``` objc
- (void)awakeFromNib {
    [[self browserTextField] setActionBlock:^{
        // do something here like launch NSOpenPanel etc.
    }];
}
```

Here, `[self browserTextField]` is a property with an IBOutlet to your custom text field.

###Creating a custom `NSTextField`###

This part is a bit like the "Making of ..." that comes with DVD's.  If you want to get under the hood and learn a little about developing for Mac OS, here's your chance.  (We've been developing so much for iOS that some of the older parts of Mac, like `NSCell` are a little foreign to us.)

Let's start from the inside out.  

####Drawing into an image####
Where possible we favor drawing into an `NSView` or in this case `NSImage` rather than loading images from the bundle.  This makes adapting to resolution changes like the Retina transition easier for us.  We use [PaintCode](http://www.paintcodeapp.com/) to generate the drawing code from vector drawings on-screen.  It's a great piece of software.

Let's take a look at the process of drawing into an image.  I'm going to focus on just that aspect rather than the Core Graphics details.  In the `CCFBrowserTextFieldButtonImage` implementation we have one public method `init`:

``` objc
- (id)init {
    self = [super init];
    if( !self ) return nil;
    
    self.size = NSMakeSize(18, 18);
    
    _drawIcon(self);
    
    return self;
}
```
The important thing to remember here is to set the size of the image before attempting to draw into it `self.size = ...`.  Then we call `_drawIcon:` on ourself.  

Drawing into the `NSImage` is a three step process.  First we lock focus on ourself.  This prepares the image for drawing.  More specifically, it sets the current graphics context to the area of the offscreen window that is used to cache the `NSImage` contents.  All of the drawing until we call `unlockFocus` is composited to the offscreen window.  In step 2, we draw the image.  I'm not going to go through each shape in this post - but you can get a feel for how the image is built up.  Finally, we unlock focus which resotores the focus to the previous owner.  That's it for drawing the image.  But the image has to go somewhere, right?  Next up, `CCFBrowserTextFieldButton`

###An embeddable button###
Next up in our inside-out tour is `CCFBrowserTextFieldButton`.  Starting with the class interface, we see block type definition:

```objc 
typedef void(^CCFBrowserButtonBlock)(void);
```
This is the typedef for the block used as an action for the button.  More on this in a minute.  Next up the class method(s):

``` objc
+ (NSSize)browserImageSize;
```
Class users will use this method to get information about the size of the image in the button.  Some of the upstream users will need to know how large our image is to provide for cursor changes and text formatting in the fields.  

Finally we have a couple methods to set the frame and/or the action handler.  On to the implementation, let's look at the `initWithFrame:` method:

```objc
- (id)initWithFrame:(NSRect)frame
{
    self = [super initWithFrame:frame];
    if( !self ) return nil;
    
    [self setButtonType:NSMomentaryPushInButton];
    [self setBordered:NO];
    [self setImage:browserImage];
    [self setImagePosition:NSImageOnly];
    [self setAutoresizingMask:NSViewMinXMargin | NSViewMinYMargin | NSViewMaxYMargin];
    
    return self;
}
```
Here we're just defining the basic properties of the button.  We want the button to be an ordinary momentary push in button, without borders, with our specified image, and positioning the image as an image only button.  So where does the image come from?  We declare some static variables in the implementation:

``` objc
static NSImage *browserImage;
static NSSize browserImageSize;
```

and we set them up in the `initialize` method:

``` objc
+ (void)initialize {
    browserImage = [[CCFBrowserTextFieldButtonImage alloc] init];
    browserImageSize = browserImage.size;
}
```

And by the way, we're using ARC throughout.  So should you.  We were reference counting  gurus before ARC; but ARC is too good to ignore.  

Finally, we have a pair of methods that set the `frame` and the `_actionBlock`  Because `NSButton` doesn't have a blocks-based action handler, we extended it by setting its `target` to `self` and the `action` to a method that just calls the action block.  On to the cell.

###A digression on why controls have cells###
Newcomers to Mac OS X are often perplexed by the existence of `NSCell`.  It's like `NSView`, but not really.  It's associated with `NSControl`, but why?

Basiclaly `NSCell` is just a lightweight view.  Take the example of `NSTableView`, in the old days, its cells were always made up of `NSCell` because the size of its instance was less than a view; and using a lighter weight object made tremendous difference in memory savings.  So, most controls are backed by an `NSCell` that handles much of the lower-level work of the control.

###So, our cell###
To take care of some drawing bits, we subclass `NSTextFieldCell`.  Here's how it works:  By overriding `-titleRectForBounds:` we have the opportunity to modify the region in which our text can be drawn.  Ordinarily the cell takes the insert area of the field to draw the text minus a little inset.  In our case, we want to give plenty of room for the button.  So we do this:

``` objc
- (NSRect)titleRectForBounds:(NSRect)bounds;
{
    NSRect buttonRect = [[self class] rectForBrowserFrame:bounds];
    float horizontalEdgeGap = 0.0f;
    
    bounds.size.width -= NSWidth(buttonRect) + horizontalEdgeGap;
    return bounds;
}
```
We get the frame of the button in our bounds, then move the width accordingly.  

###And, last but not least, `CCFBrowserTextField`###
The outermost component is the field itself.  There are a few things we need to take care of here.  For example we have to add the button itself:

```objc
- (id)_initTextFieldCompletion {
    if( !_browserButton ) {
        NSSize browserImageSize = [CCFBrowserTextFieldButton browserImageSize];
        NSRect buttonFrame = NSMakeRect(0.0f, 0.0f, browserImageSize.width, browserImageSize.height);
        _browserButton = [[CCFBrowserTextFieldButton alloc] initWithFrame:buttonFrame];
        buttonFrame = [CCFBrowserTextFieldCell rectForBrowserFrame:self.bounds];
        
        [self _setCellClass];
        
        [self addSubview:_browserButton];
        self.autoresizesSubviews = YES;
        [_browserButton setFrame:buttonFrame actionHandler:^{
            NSLog(@"pushed");
        }];
    }
    return self;
}
```

We instantiate the `_browserButton` lazily here, first getting the size of its image, then setting its frame accordingly.  We set the class of the cell our field uses directly.  More on that in a moment.  Then we add the button and set a silly default handler.  If you want the button to do something meaningful, you'll have to set the field's action handler.

Now we have to tell our custom field to use **our** cell and not the default cell for the superclass.  This is what we're doing here:

``` objc
- (void)_setCellClass {
    Class customClass = [CCFBrowserTextFieldCell class];
    
    // since we are switching the isa pointer, we need to guarantee that the class layout in memory is the same
    NSAssert(class_getInstanceSize(customClass) == class_getInstanceSize(class_getSuperclass(customClass)), @"Incompatible class assignment");
    
    // switch classes if we are not already switched
    NSCell *cell = [self cell];
    if( ![cell isKindOfClass:[CCFBrowserTextFieldCell class]] ) {
        object_setClass(cell, customClass);
    }
}

```

This private method makes use of our access to the Objective-C runtime.  In the assertion, we make sure that the instance size of our custom cell is the same as `NSTextFieldCell`.  This is just a protection against us screwing something up badly when we switch classes.  Next we do the switch using `object_setClass()`.

Enough of the swizzly stuff.  Let's look add some of the subtleties that we could have easily forgotten, like the cursor.  Over text fields, we are accustomed to seeing the I beam cursor.  But over a button, it looks silly.  So we override `resetCursorRect`.  The documentation succinctly says what it does:  "Overridden by subclasses to define their default cursor rectangles."  So, we just add the two rects with the correct cursors to the two parts of our field using `addCursorRect:cursor:`.  

That's about it for the internals of `CCFBrowserTextField`; hopefully you can put it to use in your projects. 



