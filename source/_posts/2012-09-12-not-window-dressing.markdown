---
layout: post
title: "Not window dressing"
date: 2012-09-19 10:00
comments: false
categories: [objective-c, ios, mac os, coding style]
---
After 3 years of full-time development on iOS and Mac (and many years of dabbling), I've had the chance to write and read a lot of code.  The exponential growth of these platforms has been amazing; but it has brought a number of challenges.  One of the challenges has been how to bring developers new to Objective-C and Cocoa to the community in a way that helps us all write better code.  Code that's readable and code that us mortals can maintain long after it was written.

##Broken windows##
The broken windows theory is a construct in the social sciences that deals with the establishment of norms around vandalism and other criminal behavior.  The idea is that communities that fail to care for their physical appearance are subject to more decline as the outward appearances signal an area of low value.

James Wilson and George Kelling, in the article "Broken Windows" (The Atlantic Monthly, March 1982) described this example:

>>_"Consider a building with a few broken windows. If the windows are not repaired, the tendency is for vandals to break a few more windows. Eventually, they may even break into the building, and if it's unoccupied, perhaps become squatters or light fires inside. Or consider a sidewalk. Some litter accumulates. Soon, more litter accumulates. Eventually, people even start leaving bags of trash from take-out restaurants there or breaking into cars."_

Does code suffer from the same social phenomena?  Who knows - but what developers do is a craft.  And it is a craft that contributes to a body of work, whether formalized in a discrete project or whether implicitly in the way we post code on Stack Overflow.  Ultimately, code finds itself in a body of literature that sets the standards for the community.  As craftspeople, we have a responsibility to the community to set standards that encourage clear thinking.

##Foolish thoughts##

The famous British author George Orwell wrote an essay entitled "Politics and the English Language" in 1946.  In this brief work, he describes several problems with the use of his native language and how it obscures meaning.  But the most powerful hypothesis that he puts forward is that the sloppy use of language leads to flawed reasoning:

>>_"Now, it is clear that the decline of a language must ultimately have political and economic causes: it is not due simply to the bad influence of this or that individual writer. But an effect can become a cause, reinforcing the original cause and producing the same effect in an intensified form, and so on indefinitely. A man may take to drink because he feels himself to be a failure, and then fail all the more completely because he drinks. It is rather the same thing that is happening to the English language. It becomes ugly and inaccurate because our thoughts are foolish, but the slovenliness of our language makes it easier for us to have foolish thoughts."_ (George Orwell, "Politics and the English Language", Horizon, April 1946)

Code follows this same pattern.  The manner in which we write code both reflects the way we think about a problem **and** in turn, it causes us to to frame our thoughts about the problem in the context of the solution we've crafted.  Cause and effect exert a bidirectional influence on each other.  It's clear that poor use of our language, the formal language of Objective-C, may cause us to think about the problems we're solving in faulty ways.


At Cocoa Factory, we love to code and we love to code well.  So, we're taking the next few weeks to describe our approach to readable, maintainable, functional, and beautiful code.

