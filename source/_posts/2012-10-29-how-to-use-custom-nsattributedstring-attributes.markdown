---
layout: post
title: "How to use custom NSAttributedString attributes"
date: 2012-10-29 19:36
comments: false
categories: cocoa, text, mac os
---
## How to draw a custom attribute in `NSLayoutManager` ##

The Cocoa Text System is incredibly flexible; but not nearly as well-documented as it should be given its power.  The classes and methods themselves are completely documented as is the "big picture" - but there's a lot of intermediate documentation that's missing.  

In this tutorial, we'll build an app that draws a custom attribute in an `NSTextView` like this:

![CustomAttributeTestApp](http://i.imgur.com/mErdK.png)

`NSAttributedString` is great for drawing standard attributes such as font, font size, foreground and background colors; but it gets more complicated when you need to some something that requires actual drawing.  This tutorial will show you how to do simple drawing of a custom attribute.

### Getting Started ###

Download the [example project](https://github.com/cocoa-factory/CCFCustomAttributeTutorial/zipball/master) from Github.  You need Xcode 4.5 for this project; so if you don't have it - go update Xcode first.

### `NSAttributedString` for decorated text ###

`NSAttributedString` and its mutable counterpart `NSMutableAttributedString` are used to draw decorated text.  Using these classes, you can create strings with attributes that describe how the string should look when drawn.  For example, you can add font and color attributes like this:

``` objc
NSString *sampleString = @"This is a sample string";
NSAttributedString *attributedString;
attributedString = [[NSAttributedString alloc] initWithString:sampleString
                                                   attributes:@{NSFontSizeAttribute:@24}];
```

This creates a string whose font size attribute is 24.0 pt.  And if we want to display the `attributedString` in an `NSTextView`:

``` objc
NSTextView *textView;
[textView setAttributedString:attributedString];
```

With the mutable variant `NSMutableAttributedString` you can add and remove attributes dynamically:

``` objc
NSMutableAttributedString *string;
string = [[NSMutableAttributedString alloc] initWithString:@"The quick brown fox"
              attributes:@{NSBackgroundColorAttributeName : [NSColor yellowColor]}];
```

which will render like this:

![background-color](http://i.imgur.com/qMnbA.png)

Of course, you can also combine attributes:

``` objc
NSMutableDictionary *attributes = [[NSMutableDictionary alloc] init];
    [attributes setObject:[NSColor yellowColor] forKey:NSBackgroundColorAttributeName];
    
    NSFont *font = [[NSFontManager sharedFontManager] fontWithFamily:@"Arial" traits:0 weight:5 size:24];
    [attributes setObject:font forKey:NSFontAttributeName];
    NSMutableAttributedString *string;
    string = [[NSMutableAttributedString alloc] initWithString:@"The quick brown fox"
                                                    attributes:attributes];
    [[self textView] insertText:string];
```

![Combining attributes](http://i.imgur.com/J4EWm.png)

### `NSMutableAttributedString` tracks changes to its string ###

If you want to change the underlying `NSMutableAttributedString` without disturbing its attributes, you can use its `mutableString` method to obtain an `NSMutableString` that you can manipulate behind its back, while the `NSMutableAttributedString` tracks the changes.  In fact the object you get back from `mutableString` is not actually an instance of `NSMutableString` but an instance of `NSMutableStringProxyForMutableAttributedString` instead.  This proxy object is responsible for the tracking behavior internally.

### What about custom attributes, then? ###

Let's get started building the custom attribute.  The drawing is done in the context of a layout manager - a subclass of `NSLayoutManager` Since our intent is to use our custom attribute in the context of an `NSTextView` we should look at the architecture of that class first.  `NSTextView` has a single text container in which is lays out text.  The `NSTextContainer` is a rectangular region in which to layout text.  Each `NSTextView` has a default text container, but it is possible to replace the text container using the `replaceTextContainer` method.  The text container uses a layout manager to layout and draw the text.  There is readonly access to the text container's layout manager on `NSTextView`.  In order to give `NSTextView` a new layout manager, we have to set it on a new `NSTextContainer` object.

So let's start with a custom text view that we'll call `CCFTextView`.  You can find the source code in the "view" folder.  This text view basically does on thing - replace its `NSLayoutManager`

``` objc
static CCFTextView *commonInit(CCFTextView *self) {
    //  set up our initial text size
    NSFont *font = [[NSFontManager sharedFontManager] fontWithFamily:@"Helvetica" traits:0 weight:5 size:24.0];
    NSDictionary *attributes = @{NSFontAttributeName : font};
    [self setTypingAttributes:attributes];
    
    //  replace our layout manager with custom layout manager
    CCFCustomLayoutManager *layoutManager = [[CCFCustomLayoutManager alloc] init];
    NSTextContainer *textContainer = [[NSTextContainer alloc] init];
    
    [self replaceTextContainer:textContainer];
    [textContainer replaceLayoutManager:layoutManager];
    return self;
}
```

The `commonInit` function is called from either `initWithCoder:` or `initWithFrame:` so that no matter how the `CCFTextView` gets initialized, we replace its text container's layout manager with our own subclass.  Let's look at the `NSLayoutManager` subclass - `CCFCustomLayoutManager` in the "helpers+managers" directory.  In the header file "CCFCustomLayouManager.h" we define a few constants.  `CCFSpecialHighlighterAttributeName` is the name of our custom attribute and `CCFHighlightColorKey` and `CCFLineColorKey` are keys to the dictionary value of our attribute.

In the implementation of our layout manager, we override a single method `drawGlyphsForGlyphRange:atPoint:`.  Here we'll digress about glyphs vs. characters.

### Glyphs versus characters ###

The **character** can is the smallest unit of a written language that has meaning.  In Roman and other alphabets, it maps to a particular sound in the spoken counterpart of the written language.  However in the case of other languages, like Chinese, it can represent an entire word.

A **glyph** on the other hand is a graphically-concrete form of a character.

![Glyphs-vs-characters](http://i.imgur.com/3z2M5.png)

The distinction is important, because while we're manipulating characters in our code, the text system is working behind the scenes laying out glyphs, not characters.  In this case, we need to do both.  That's why out `NSLayoutManager` subclass overrides `drawGlyphsForGlyphRange:atPoint`.  So let's look a little more closely at what we do in this method, which we'll build up from pseudo-code

``` objc
- (void)drawGlyphsForGlyphRange:(NSRange)glyphsToShow atPoint:(NSPoint)origin {
    /*
    	iterate over our glyph ranges
    	map the glyph range back to the character range that it represents
    	check if our attribute is set on the character range
    	if attribute is set
    		get the rect of where we should draw the glyph
    		do our drawing
    */
}
```

First, since we need to refer to the character sequence when we do the mapping, we need a source for that mapping.  Fortunately, `NSLayoutManager` keeps a reference to its `NSTextStorage` object.  This object is a subclass of `NSMutableAttributedString`.  We will get this reference and copy `glyphsToShow` to a local variable so that we can iterate over its span.

``` objc
- (void)drawGlyphsForGlyphRange:(NSRange)glyphsToShow atPoint:(NSPoint)origin {
	NSTextStorage *textStorage = self.textStorage;
    NSRange glyphRange = glyphsToShow;
    while (glyphRange.length > 0) {
    	/*
    	map the glyph range back to the character range that it represents
    	check if our attribute is set on the character range
    	if attribute is set
    		get the rect of where we should draw the glyph
    		do our drawing
    	*/
	}
}
```

Now, we take care of the glyph-to-character mapping:

``` objc
- (void)drawGlyphsForGlyphRange:(NSRange)glyphsToShow atPoint:(NSPoint)origin {
	NSTextStorage *textStorage = self.textStorage;
    NSRange glyphRange = glyphsToShow;
    while (glyphRange.length > 0) {
    	NSRange charRange = [self characterRangeForGlyphRange:glyphRange actualGlyphRange:NULL];
        NSRange attributeCharRange, attributeGlyphRange;
        id attribute = [textStorage attribute:CCFSpecialHighlightAttributeName
                                      atIndex:charRange.location longestEffectiveRange:&attributeCharRange
                                      inRange:charRange];
        attributeGlyphRange = [self glyphRangeForCharacterRange:attributeCharRange actualCharacterRange:NULL];
        attributeGlyphRange = NSIntersectionRange(attributeGlyphRange, glyphRange);
    	/*
    	check if our attribute is set on the character range
    	if attribute is set
    		get the rect of where we should draw the glyph
    		do our drawing
    	*/
	}
}
```

Then to check if the attribute is set on this `charRange`, we just test for non-nil:

``` objc
- (void)drawGlyphsForGlyphRange:(NSRange)glyphsToShow atPoint:(NSPoint)origin {
	NSTextStorage *textStorage = self.textStorage;
    NSRange glyphRange = glyphsToShow;
    while (glyphRange.length > 0) {
    	NSRange charRange = [self characterRangeForGlyphRange:glyphRange actualGlyphRange:NULL];
        NSRange attributeCharRange, attributeGlyphRange;
        id attribute = [textStorage attribute:CCFSpecialHighlightAttributeName
                                      atIndex:charRange.location longestEffectiveRange:&attributeCharRange
                                      inRange:charRange];
        attributeGlyphRange = [self glyphRangeForCharacterRange:attributeCharRange actualCharacterRange:NULL];
        attributeGlyphRange = NSIntersectionRange(attributeGlyphRange, glyphRange);
    	
    	if( attribute != nil ) {
    		/*
    			get the rect of where we should draw the glyph
    			do our drawing
    		*/
		}
   	}
}
```

Finally, the drawing is the easiest part.  We just need to bracket our drawing code with calls to save then restore the `NSGraphicsContext` before drawing.  To get the rectangle in which our glyph is drawn, we ask for the `boundingRectForGlyphRange:inTextContainer:`.  Lastly, we have our completed implementation:

``` objc
- (void)drawGlyphsForGlyphRange:(NSRange)glyphsToShow atPoint:(NSPoint)origin {
    NSTextStorage *textStorage = self.textStorage;
    NSRange glyphRange = glyphsToShow;
    while (glyphRange.length > 0) {
        NSRange charRange = [self characterRangeForGlyphRange:glyphRange actualGlyphRange:NULL];
        NSRange attributeCharRange, attributeGlyphRange;
        id attribute = [textStorage attribute:CCFSpecialHighlightAttributeName
                                      atIndex:charRange.location longestEffectiveRange:&attributeCharRange
                                      inRange:charRange];
        attributeGlyphRange = [self glyphRangeForCharacterRange:attributeCharRange actualCharacterRange:NULL];
        attributeGlyphRange = NSIntersectionRange(attributeGlyphRange, glyphRange);
        if( attribute != nil ) {
            [NSGraphicsContext saveGraphicsState];
            
            NSColor *bgColor = [attribute objectForKey:CCFHighlightColorKey];
            NSColor *lineColor = [attribute objectForKey:CCFLineColorKey];
            
            NSTextContainer *textContainer = self.textContainers[0];
            NSRect boundingRect = [self boundingRectForGlyphRange:attributeGlyphRange inTextContainer:textContainer];
            
            [bgColor setFill];
            NSRectFill(boundingRect);
            
            NSRect bottom = NSMakeRect(NSMinX(boundingRect), NSMaxY(boundingRect)-1.0, NSWidth(boundingRect), 1.0f);
            [lineColor setFill];
            NSRectFill(bottom);
            
            NSRect topRect = NSMakeRect(NSMinX(boundingRect), NSMinY(boundingRect), NSWidth(boundingRect), 1.0);
            NSRectFill(topRect);
                       
            [super drawGlyphsForGlyphRange:attributeGlyphRange atPoint:origin];
            [NSGraphicsContext restoreGraphicsState];
        }
        else {
            [super drawGlyphsForGlyphRange:glyphsToShow atPoint:origin];
        }
        glyphRange.length = NSMaxRange(glyphRange) - NSMaxRange(attributeGlyphRange);
        glyphRange.location = NSMaxRange(attributeGlyphRange);
    }
}
```

### Setting attributes ###

Let's turn our attention to `CCFMainWindowController` where our attributes are being managed.  When the user presses the highlight button, we want to tell the text view to apply our attribute to the selection - which is what we do in `setCustomAttribute:`:

``` objc
- (IBAction)setCustomAttribute:(id)sender {
    //  add our custom attribute to the selected range
    NSRange selectedRange = [[self textView] selectedRange];
    NSTextStorage *textStorage = self.textView.textStorage;
    [textStorage addAttribute:CCFSpecialHighlightAttributeName value:[self attributeColors] range:selectedRange];
}

#pragma mark - Private

//  return dictionary of highlight and line colors for our custom attribute's value
- (NSDictionary *)attributeColors {
    return @{  CCFHighlightColorKey : self.highlightColorWell.color, CCFLineColorKey : self.lineColorWell.color };
}

```

The rest of the code in `CCFMainWindowController` is for setup and for observing for changes in the highlight and line colors.  Using Key-value observing, we are able to detect when the colors change and re-do our markup accordingly.

Athough here's much more to the text system in Cocoa you should have a good starting point for custom attributes. 