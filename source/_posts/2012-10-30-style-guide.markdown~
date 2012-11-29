---
layout: post
title: "Cocoa Factory Style Guide"
date: 2012-10-30 05:00
comments: false
categories: coding practice
---
## Style guide ##

A consistent coding and project layout style is one of the keys to writing readable and maintainable software.  As has been pointed out before, even the solo developer works in a team along with his future self.  And your future self will thank you for writing readable code.

Here's our style guide.  Many of the choices are arbitrary. The placement of braces, for example, follows a common - but by no means universal - convention.  But, arbitrary or not here's how we organize our code.

### Project organization ###

There are two layers of project-level organization that we deal with - **IDE-level organization** and **file-system level organization**.  Although Xcode is capable of organizing the project to any degree of granularity you want, relying _solely_ on Xcode to organize the project leads to a catastrophic mess in the filesysem  with all of the classes, header files, and resources living in a jumbled mess.

Therefore, we begin a project with a basic degree of organization in the file system that is reflected in Xcode and provides a skeletal organization for the project as it develops.

{% img left http://i.imgur.com/SG3Uf.png 'file system organization' %}  Before writing any code, we add the directories as depicted to the file system.  Any project template files that are generated automatically by Xcode are then moved into the appropriate directories in the file system.  At that point, we return to Xcode and delete the file references.  Then we drag all of the folders from the file system back into Xcode.  This establishes the basic structure in Xcode that parallels that in the file system.  As our project grows, we add the remaining hierarchical structure only in Xcode.  We adopted this practice after reading a post by Adrian Kosmacezewski entitled [Code Organization in Xcode Projects](http://akosma.com/2009/07/28/code-organization-in-xcode-projects/).  The approach he describes is for Xcode 3; but it works fine with modern versions of Xcode.  The work of setting up the initial organization needs to be done up-front before you write any code.  Otherwise, moving files into place in the file system becomes tedious.

{% img left http://i.imgur.com/EZMkz.png 'Xcode organization' %}

### Naming conventions ###

We adhere nearly completely to the Apple conventions on naming [generally](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingBasics.html#//apple_ref/doc/uid/20001281-BBCHBFAH) and for [methods](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingMethods.html#//apple_ref/doc/uid/20001282-BCIGIJJF), [functions](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingFunctions.html#//apple_ref/doc/uid/20001283-BAJGGCAD), and [properties](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CodingGuidelines/Articles/NamingIvarsAndTypes.html#//apple_ref/doc/uid/20001284-BAJGIIJE)

Our company prefix is `CCF` for Cocoa Factory.  All public class names begin with the `CCF` prefix.  For non-public subclasses of class clusters, we use `_CCF` for the class prefix.

### Typography ###

We use Menlo 11 pt typeface for coding in Xcode.  The syntax highlighting theme is [Spacedust](https://github.com/mhallendal/spacedust-theme).  

All other typographical considerations follow Apple's guidelines.  In other words, we use camel case without underscores or dashes to separate words.

``` objc
BOOL canProvideData     //	CORRECT
BOOL can_provide_data   //	INCORRECT

BOOL PNGIsValid         //	CORRECT (well-known abbreviations can be capitalized)
```

### Organization of the implementation file ###


#### Indentation ####

Indentation is 4 spaces per level.

#### Top-down ####

We begin with a standard header block:

``` objc
/**
 *   @file NAME_OF_FILE
 *   @author WHO_WROTE_THE_FILE (www.cocoafactory.com)
 *
 *   @date DATE_AND_TIME in this format: 2012-10-29 05:56:53
 *
 *   @note Copyright 2012 Cocoa Factory, LLC.  All rights reserved
 */
```

Next, any files that should be imported.  The files should be listed in the following order:

1. The **class header** file
2. Any **framework** headers that are required
3. **Model** headers
4. **Controller** headers
5. **Helper/Manager** headers
6. **View** headers

After the `#import` statements, skip one line then declare any `#define` statements, followed by any static variable then static function declarations.  If the class declares a class extension, then it should appear next just before the `@implementation` block.

#### Pragma marks ####

We use `#pragma mark` to separate source code into sections.  The types of tags used will vary; but the following are standard:

``` objc
#pragma mark - Object lifecycle
#pragma mark - View lifecycle       //  for view controller classes
#pragma mark - KVO                  //  key-value observing
#pragma mark - Interface actions    //  all IBActions go here
#pragma mark - Public               //  all public methods go here
#pragma mark - Private              //  all private methods go here
#pragma mark - NSTableViewDelegate  //  each protocol that the class adopts gets its own #pragma mark block
```

#### Private methods ####

Do not use an underscore to indicate a private method.  We name private methods using the same conventions as public methods.  _(NOTE: We are in the process of changing older code to meet this guideline.  For years, we ignored Apple's waning; but worries about collision finally compelled us to change.)_


### Braces ###

We follow the Kerninghan and Ritchie (K & R) style of brace placement with the `else` statement getting its own line:

``` objc
- (void)foo {
    if( flag ) {
        //  do something
    }
    else {
        // do something else
    }
}

```

### Parenthesis ###

The opening parenthesis touches its operator and is followed by a space.  The closing parenthesis is preceded by a space:

``` objc
//  CORRECT
if( flag ) {
	// do something
}

//  INCORRECT
if (flag) {
	//  do something
}
```

### Instance variables ###

Instance variables that are explicitly declared should have a preceding underscore character.

All instance variables are declared in the implementation file _**unless**_ they are used by a subclass.  In the latter case, all of the instance variables should be declared in the class interface file, thereby keeping all of the ivars together.

### Protocols ###

Protocols are always declared in their own header file.

### Method signatures ###

The method signature should be left-justified with one space after the scope (the + or -).  There should be a space after each segment of the method and a space between the object type and the pointer * for method arguments.

``` objc
- (void)setFoo:(NSString *)aFoo;    //  CORRECT

-(void)setFoo:(NSString *)aFoo;     //  INCORRECT - no space after the scope
- (void)setFoo: (NSString *)aFoo;   //  INCORRECT - space after the colon
- (void)setFoo:(NSString*)aFoo;     //  INCORRECT - no space between type and pointer *
```

