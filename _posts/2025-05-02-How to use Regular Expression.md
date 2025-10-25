---
title: How to use Regular Expression
description: Discover how a Network Engineer embarks on a beginner-friendly journey to learn Git—exploring its potential to adapt to a changing tech landscape, document key lessons, and troubleshoot effectively in this first post of an insightful series.
date: 2025-05-02 12:00:00:01 +/-0000
categories: [linux, Regex]
tags: [sed,grep]     # TAG names should always be lowercase
toc: true
comments: true
image:
  path: '/assets/img/headers/regex.webp'
  alt: 'Terminal with the text /(reg)ex/'
pin: false
published: true
---
## Scenarios

Regular expression is a cool tool used to grab outputs.  I use it all the time when working with Cisco Gear.  When updating tickets, or wanting to verify just a few outputs, I use regular expression to gather details from devices that.  This comes in real handy if you only what specific details and not need the entire output.  This blog is diving a bit deeper into efforts of learning Regex.  

The interesting part to this, is that Cisco and Linux, and Solarwinds, and other tools all use different forms of regular expression.  Hopefully the below will provide some good notes to review and understand how to use Regex in my progress to learn Automation and and Pyats.


## Basics
This searches for anything with"express" in the document
```bash
grep express /usr/share/words.txt
```

How to find specific text in a directory and list the folder/file it exists
```bash
find /usr/share/doc/folder -exec grep -H regular {} \; 2> /dev/null
```
- **find** - feature called find
- **/usr/share/doc/folder** - Folder you will be completing find
- **-exec** - Execute the following command
- **grep** - User grep
- **-H** - Used to display location filename
- **regular** - This is the regular expression you will use to search the folder
- **{}** - This is where output of the search is placed I guess
- **/\;** - This closes the exec listed above
- **2>** - Ignores some errors (Try running it without and see the output)
- **/dev/null** - This additionally inores this folder? Not sure why since you are searching in this folder



### sed
"Sed" is a stream editor that edits in place without actually having to use nano or vi. 

```bash
sed -i 's/zero/ZERO/g' words.txt
```
- ***sed*** - use the sed application
- ***-i*** - inline modify
- ***s*** - substitute this for that (replace)
- ***zero*** - This section is where you put your relar expression
- ***ZERO*** - This is the output that you want to replace it with
- ***g*** - Updates all instances in the file except just the first (***global***)
- ***words.txt*** - Define the file you want to complete the substituation for

> `sed` uses `\` to inform the start and finish of regular expression.  `grep` does not require this. 
{: .prompt-tip }


### grep
Example: How to grep an entire file for the word hello
```bash
grep hello file.txt <-- just current working directory
grep hello *  <-- all files in current directory
```

## Grep Flavors
1. BRE - Base Regular Express (grep)
2. ERE - Extended Regular Expressions (egrep) - Slightly more functionality
3. Emacs - Macbooks
4. ViM - Vim text editor has there own
5. PCRE - Perl Compatible Regular Expressions (Python, Javascript) (grep -P) - This is what most will use
6. Perl has there own, but won't use since i don't use Python

### Flavor Summary
- grep - BRE
- egrep uses ERE
  - egrep is equivalent to grep -E
- grep -P - Most common and most features supports Javascript and Python
  - https://remram44.github.io/regex-cheatsheet/regex.html


### Either Or |
When I was first introduced to regular expression I didn't have to worry about "***escape***" characters.  This means that you are using an output that is normally use in programming.  While using the Either/Or bar, "***|***" in the below statement, you are actually looking for "***hello|goodbye***".  Not ***hello*** or ***goodbye***.
```shell
grep 'hello|goodbye' /usr/share/dict/words
```

To get around this, you need to use the escape character ***\\***.
```shell
grep 'hello\|goodbye' /usr/share/dict/words
```

> One way to work around not using the `escape character` is to use `egrep` (extended regular expression)
{: .prompt-tip }

```bash
egrep 'hello|goodbye' /usr/share/dict/words
or
grep -E 'hello|goodbye' /usr/share/dict/words
```

## More Matching case senstive -i, -f, -v
Regular expression is case senstive if you want to ignore case you can use the ***-i*** here. (ignoring case)
```bash
grep -i 'austr' /usr/share/dict/words
```

You can create a file that has regular expression in it. You can call it to match files somewhere else. 
regex.txt <-- this is a file created with regex in it to match

So below statement takes the file regex.txt with the regular expression in it, and matches the directory or file words.
```shell
grep -f regex.txt /usr/share/dict/words
```
- -f = filename used so we are calling a created file "regex.txt

Below example looks in the file words.txt and provides the lines that do not have "***ne***" in it. 
```shell
grep -v 'ne' words.txt
```

### Basic Syntax Topics . ^ and $
- ***.*** - any character except ***new*** line character
- ***^*** - match beginning of string
- ***$*** - match the end of a string (or right before newline at the end of the string)

### Sillyness with "." and escape character
If you want to match an actual period (a.b.c) you need to do the following
```shell
grep a\.b\.c file.txt
```

If you want to match something that begins with ^ and ends with & you need to do the following
```shell
grep ^\^\$$ file.txt
```
- The first ^ is the start
- The \\^ uses the escape character to use the actual "^"
- The \\$ uses the scape character to use the acutal "$"
- THe $ at the end marks the end of the file. 

So the below should match anything that looks like this "^$" which is a very rare occurance to shares the example on how to use escape characters. 

## Using grep -P  < Uses PCRM most feature rich

```shell
grep -P n words.txt
```

## Regex Rule #1 - The first match wins
"a ***fun***damental idea of regexes: they are ***fun***"

  - The first occurance matches.

## Classes - match a character amongst a group []
Match ***only*** one character, either [a,b,c,d, or e]
```shell
/[abcde]/  <- Matches one character (lowercase), either a,b,c,d, or e
/[a-e]/    <- Same as above uses a range
/[a-z]/    <- Match one lower case alpha
/[0-9]/    <- match one digit character
```

Working with above you can play with -
```shell
grep -P [a-z-][0-9] text.txt
or 
grep -P [a-z\-][0-9] text.txt  <- uses the escape character not needed though. this is more clarity
```
The above matches anything a through z ***or*** dash(-) followed by a number 0 - 9.

so tes***t1***, tes***t1***23, one***-1***, two***-2*** all match

### Phone Number match
```shell
grep -P '[0-9][0-9][0-9][ \-][0-9][0-9][0-9][ \-][0-9][0-9][0-9]' phonenumbers.txt
```

### Not in a class [^regex]
If the Class begins with ***^*** then it means should not be in the class
```shell
/[^abcde]/ - Match one characther, neither a,b,c,d, nor e
/[^a-z]/   - Same - non lower alpha character no matched "***a***" through "***z***"
/[^0-9]/   - matche one non digit character (any character not a number
```
Example of matching a non number followed by a non lowercase alpha
```bash
grep -P [^0-9][^a-z] text.txt
```
Above finds examples like tes***t1***23 or tw***0-***2

## POSIX Character Classes
These are some character classes that are common but weird syntax

- /[a-z]/
- /[[:lower:]]/
Outer brackets create the class
Inner brackes represent the character class i.e. lower

### POSIX NEGATE (not match)
If you want to do the not matching use the ***^***
- [[:^lower:]] - Anything that doesn't match lowercase character

Another example
- /[12[:^digit:]]/

This matches anything that has 1 or 2 followed by a non digit.
So basically matches anything that doesn't have a 0 or 3 through 9

Another Example
- /[abc[:^lower:]]/

This matches anything that has a,b, or c, and no lower case.

### Other POSIX
  most never used
- alnum - letters and digits [a-zA-Z0-9]
- alpha - letters [a-zA-Z]
- ascii - ascii codes 0 - 127
- blank - space or tab [ \t]
- cntrl - control characters
- digit - digits [0-9]
- graph - printing characters, excluding space
- lower - lower case letters [a-z]
- print - printing characters, including space
- punct - printing chars, excl letters, digits, space
- space - white space [ /t/n/f/r] and VT
- upper - upper case letters [A-Z]
- word - word characters [a-zA-Z0-9_]
- xdigit - hex digits

## Generic Character Classes (mostly use)
Character classes used all the time that have shorthand
- \d - digit [0-9]
- \D - non-digit [^0-9]
- \w - word [a-zA-Z0-9_]
- \W - non-word [^a-zA-Z0-9_]
- \s - space character [ \t\n\r\f] (space,tab,newline,return,formfeed -next page)
- \S - non space [^ \t\n\r\f]
- \H - non-horizontal white space
- \v - vertical white space
- \V - non-vertical white space

Now with these new shortcuts we can shorten looking for phone numbers
Before:
```shell
grep -P '[0-9][0-9][0-9][ \-][0-9][0-9][0-9][ \-][0-9][0-9][0-9]' phonenumbers.txt
```
After:
```shell
grep -P '\d\d\d[ \-\.]/d/d/d[ \-\.]/d/d/d/d' phonenumbers.txt
```

More examples
 - [A-Z][a-z] - Capital letter followed by lower case letter "One"
 - \w\s\d - match word space then digit
 - [[:upper]] - match one occurance of an upper case letter
 - ^[[:upper]] - Match beginning of line starts with upper letter
 - [[:alpha]]& - Match where the last letter is not an alpha letter
 
 ## Quantifiers (like phone numbers)
  - \* - zero or more
  - \+ - one or more
  - \? - zero or one

  ab*c - "a", zero or more "b", "c"
  - The * points to the thing before it and repeats as many times. (ac, abc, abbd, abbbc ...)

  ab+c - "a", one or more "b", "c"
  -  (abc, abbc, abbbc)

  ab?c - "a", zero or one "b", "c"
  - (ac or abc)

{n} - n times
ab{3}c - "a",3 "b", "c"
- abbbc

{n,} - n or more
ab{3,}c - "a", three or more "b", "c"
 - abbbc, abbbbc, abbbbbc, abbbbbbc, ...

 {n,m} - n through m
 ab{3,5}c - "a", 3,4,or 5 "b", "c"
 - abbbc, abbbbc, or abbbbbc



<!--Stopped at 124 minute -->


## Macbook install grep
```shell
brew update
brew install grep
```
The improved ***grep*** is called ***ggrep***

## Helpful online tool to work with regex
1. https://regexr.com
2. https://regex101.com
3. https://www.regextester.com
4. https://www.freeformatter.com/regex-tester.html
5. https://regexcrossword.com



