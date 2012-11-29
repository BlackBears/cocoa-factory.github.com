---
layout: page
title: "Regex cheat sheet"
date: 2012-11-28 11:40
comments: false
sharing: true
footer: true
---
### Regular expression syntax ###

#### Anchors ####

Element	| Explanation
----------	| -----------
`^`			|Start of a string or start of line if multiline option is enabled
`\A`		|Start of string
`$`			|End of string or end of line if multiline option is enabled
`\Z`		|End of string
`\b`		|Word boundary
`\B`		|Not word boundary
`\<`		|Start of word
`\>`		|End of word

#### Character classes ####

Element	| Explanation
----------	| -----------
`\c`		|Control character
`\s`		|White space
`\S`		|Not white space
`\d`		|Digit
`\D`		|Not digit
`\w`		|Word
`\W`		|Not word
`\x`		|Hexadecimal digit
`\O`		|Octal digit

#### Assertions ####

Element	| Explanation
----------	| -----------
`?=`		|Lookahead assertion
`?!`		|Negative lookahead
`?<=`		|Lookbehind assertion
`?!=`		|Negative lookbehind
`?>`		|Once only
`?()`		|Condition [if then]
`?()|`		|Condition [if then else]
`?#`		|Comment

#### Quantifiers ####

Element	| Explanation
----------	| -----------
`*`			|0 or more
`+`			|1 or more
`?`			|0 or 1
`{3}`		|exactly 3
`{3,}`		|3 or more
`{3,5}`	|3,4 or 5
Add `?`	|to make ungreedy

#### Groups and Ranges ####

Element	| Explanation
----------	| -----------
`.`			|Any character except new line `\n`
`(a|b)`	|a or b
`(..)`		|Group
`(?:..)`	|Noncapturing group
`[a-c]`	|Range a,b or c
`[^abc]`	|Not a,b or c
`[A-Z]`	|capital letters A to Z
`[0-9]`	|Digits 0 to 9
`\1`		|First group or subpattern


