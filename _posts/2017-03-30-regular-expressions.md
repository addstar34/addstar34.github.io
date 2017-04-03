---
layout: post
title: Regular Expressions
author: Adam D
---

When I first learnt about regular expressions I thought “cool I can match characters or patterns in text, I’ll just google the regex pattern when I need it” and when that time came I discovered regex is almost like a programming language itself with many different concepts like; anchors, alternation, groups, backreferences, lookahead, lookbehind and the list goes on…

As there are so many regex concepts and engines I decided to write this blog and just briefly cover most of them. I am also writing this with relation to Ruby version 2.0+ which uses the Onigmo engine.

# Regular expression engines

There are many different regular expression implementations. This means some features/concepts might not work or might function slightly differently depending on which engine you’re using. For example Ruby 2.0 and onwards uses the Onigmo engine which is a fork of the Oniguruma engine. Atom and Sublime IDE’s also use Oniguruma so the experience across Ruby and these editors will be very similar, although the Onigmo fork focuses on supporting new features. But then Javascript’s regex engine doesn’t support as many features such as no support for lookbehind, no atomic groups, no possessive quantifiers. Although Javascript's regular expression engine is part of the ECMA-262 standard so at least the different implements of Javascript (v8, chakra, spidermonkey) should all experience the same regular expressions.

# Defining a regular expression

Defining a regular expression is commonly done inside forward slashes such as `/regex/`. In ruby you can also use `%r{regex}` or the `Regexp::new` constructor.

# Character types

**Literal characters** simply match the character itself `a` will match `a`, `9` will match `9`.

**Special characters** also known as metacharacters have special meanings. The special characters are `\ ^ $ . | ? * + () [ {`

To use **special characters** as literal characters they need to be escaped by preceding them with `\` for example to use `\?` will match `?`

**Non printable characters** such as tab `\t`, carriage return `\r`, line feed `\n` and line separator in windows `\r\n` can also be matched.


# How a regex engine works

Understanding how the regex engine works internally is important and will help with reading and constructing regular expressions.

The regex engine will start with the first regex token and start walking through the string looking for a match. If there is a match the regex engine moves to the next token and continues looking for a match. If a match fails the engine will backtrack and then try to find another way to make a match. As soon as the regex engine matches all tokens it will return that match even if there is a better match later in the string.

# Character classes

Character classes are defined using `[]` and match one from the selection of characters within.

`[a]` matches `a`  
`[ab]` does not match `ab` it matches `a` or `b`

# Negated character class

A negated character class is defined using `^` immediately after the opening `[` and this matches any character that is *not* in the character class.

`[^a]` matches any character that is *not* `a`

# Special Characters in character classes

When using special characters within a character class most are taken as literal characters except for `]`, `\`, `^` and `-`.

`[+]` matches `+`  
`[*]` matches `*`

`[\^]` backslash escapes the carrot and will match `^`  
`[\]]` backslash escapes the closing square bracket and will match `]`  
`[a-f]` the hyphen is used for ranges, matches one lowercase letter from `abcdef`  
`[1-4]` number range matches one digit from `1234`  

# Shorthand character classes

There are a number of shorthand characters and these can be used inside and outside of character classes.

`[\d]` is shorthand for any digit and matches `[0-9]`  
`[\w]` is word character and matches ASCII characters `[A-Za-z0-9_]`  
*note that \w includes digits and can also include other characters like the underscore*  

`[\s]` is whitespace character and matches `[ \t\r\n\f]`

# Negated shorthand character classes

There are shorthand ways to do negation as well instead of using the `^`.

`[\D]` is not a digit and is shorthand for `[^\d]`  
`[\W]` is not a word character and is shorthand for `[^\w]`  
`[\S]` is not a whitespace character and is shorthand for `[^\s]`

# Extra shorthand character classes

There are more character classes but they can vary depending on the regex engine being used. For example the Onigmo engine used by Ruby has:

`\h` is hexadecimal-digit character which matches `[0-9a-fA-F]`  
`\H` is non-hexadecimal-digit character

# Character class subtraction

Character class subtraction is used to subtract one character class from another. It is defined as `[...-[...]]`

`[abcd-[d]]` the `d` is subtracted from `abcd` making it match `a`, `b` or `c`
`[a-z-[aeiou]]` matches any lowercase consonant

Character class subtraction can also be nested which will subtract from the subtraction.

`[abcde-[cde-[c]]]` will subtract `c` from `cde` which leaves `de` to be subtracted matching `a`, `b` or `c`

Negation can also be used with character class subtraction and it takes precendence before the subtraction.

`[^abcd-[de]]` this means `not abcd` so any other character subtract `de` which matches any character `not abcde`

The character class subtraction always needs to be the last token within the character class.


# Character class intersection

Character class intersection matches a character that is present across multiple character sets. It is defined as `[...&&[...]]` or in Ruby the inside square brackets can be left out if not using negation like this `[...&&...]`

`[abc&&cde]` this only has `c` present in both sides so it will match `c`
`[abcd&&abc&&ab]` this only has `ab` present across all intersects so it will match `a` or `b`

When using negation with character class intersection with the onigmo engine the intersect takes precedence over the negation.

`[^abc&&cde]` so `abc` AND `cde` only has `c` in both present so is the same as `[^c]` matching any character `not c`

But if you put the negation on the right hand side you need to use `[]` and this will set the precedence to the negation then the intersect.

`[abc&&[^cde]]` is `not cde` leaving only `ab` in both sets so this will match `a` or `b`

# Dot

The `.` character matches anything except for a line break `\n`

An alternative to using dot is to use a negated character class, and this is probably the better option most of the time.

# Anchors

Anchors are used to match a position. `^` matches the position at the beginning of the line, including after a line break where a new line starts. `$` matches the position at the end of the line and before a line break. `\A` matches the beginning of the string, *not* after line breaks. `\Z` matches the end of the string, or before newline at the end and `\z` matches the end of the string.

*When validating input its probably a good idea to remove leading and trailing whitespace.*

# Word boundaries

A word boundary allows you to match whole words. It is defined using `\b`. It is an anchor so it matches a position and there are 3 positions it can match.

Before the first character where the character of `\w`
After the last character where the character matches `\w`
Inbetween 2 characters if 1 matches `\w` and the other doesn't

The negated version for the word boundary, not a word boundary, is `\B`

# Alternation

Alternation matches one regular expression out of multiple regular expressions. It is defined using `|`

`hello|hi|hey` matches `hello` or `hi` or `hey`

To limit the alternation use brackets.

`\b(hello|hi|hey)\b` is similar to the above but it matches only whole words

Alternation stops as soon as it finds a match so order can matter.

# Quantifiers

Quantifiers set the quantity for the previous regex token.

`*` matches zero or more times
`+` matches one or more times
`?` matches one or none
`{3}` matches exactly three times
`{3,}` matches three or more times
`{3, 5}` matches three to five times

Quantifiers are `greedy` meaning they try to match the token as many times as possible and they give up matches gradually as the engine backtracks.

Quantifiers can be made `lazy`, also known as `reluctant`, meaning they try to match the token as few times as possible and they gradually expand matches as the engine backtracks. To make a quantifier lazy add a `?` to it.

`*A` matches `AAA` in `AAA`
`*A?` returns an `empty` match in `AAA`
`\d+` matches `12345` in `12345`
`\d+?` matches `1` in `12345`
`\w{2,4}` matches `abcd` in `abcd`
`\w{2,4}?` matches `ab` in `abcd`

Quantifiers can also be made `possessive` they are `greedy` as they try to match the token as many times as possible but they don't give up matches when its time for the engine to backtrack. Possessive quantifiers are useful for speeding up regex performance but they can change what is matched.

# Grouping

Using `()` creates a group and allows you to apply a quantifier to the group or restrict alternation.

`()` are also numbered capturing groups which store the match allowing it to be referenced later, called a backreference, and match something that was previously matched.

`([abc])\1` can match `aa`, `bb` or `cc`
`([abc])\1\1` can match `aaa`, `bbb` or `ccc`
`([ab])([yz])\1\2` can match `ayay`, `azaz`, `byby` or `bzbz`

Backreferences can also be relative using `\k<-1>`

`(a)(b)(c)\k<-1>` matches `cc`
`(a)(b)(c)\k<-2>` matches `bb`

You can disable the numbered capturing by by putting a `?:` immediately after the opening '('

`(?:[abc])` matches `a`, `b` or `c` (there is no backreference available)

When backtracking into captured groups happens if a new match is found it will overwrite the previously stored match.

Instead of numbered groups you can used name groups using `?<name>` to define the name and `\k<name>` for the backreference.

`(?<this>[abc])\k<this>` can match 'aa', 'bb' or 'cc'

There is also another sort of group called an atomic group. An atomic group won't backtrack into the group once its exited the group. An atomic group is defined using `?>` immediately after the opening `(`

`a(bc|b)c` normal group that matches `abcc` and will also match `abc` using backtracking
`a(?>bc|b)c` atomic group that only matches 'abcc' because once it exists the group there is no backtracking so the alternation in the group won't be attempted

Groups and backreferences cannot be used inside character classes doing so will cause them to be used as literal characters.

# Mode modifiers

Mode modifiers are like extra options to control how a pattern can match. These can be passed in after the regex `/regex/modifier`.

`/abc/i` the `i` tells the regex to `ignore case` so this can match regardless of the case eg `aBc`

A mode modifier can also be turned on/off within the regular expression.

`/a(?i:b)c` can match `aBc`

Ruby supports the follow mode modifiers:

`/regex/i` ignore case  
`/regex/m` enable dot `.` to match newline characters  
`/regex/x` ignore whitespace and comments in pattern  
`/regex/o` perform #{} interpolation only once

# Free-spacing mode and comments

Using the `x` mode modifier mentioned means literal white space in a regex will be ignored as well as allow comments to be added using `#`

`/[1-9][0-9]{3}[abc]/x` this could be written like so:

`/[1-9] # match a digit 1 to 9`  
`[0-9]{3} # match exactly 3 digits 0 to 9`  
`[a b c] # match a b or c`  
`/x`

# Lookarounds

Lookarounds are zero-length assertions meaning they will `lookaround` around, either `lookahead` or `lookbehind` the current character and assert if there is a match or not without actually moving onto this character. There are 4 lookarounds:

Lookahead `(?=a)` means assert what immediately follows the current position is `a`
Lookbehind `(?<=a)` means assert what immediately precedes the current position is `a`
Negative lookahead `(?!a)` means assert what immediately follows the current position is *not* `a`
Negative lookbehind `(?<!a)` means assert what immediately precedes the current position is *not* `a`

# Conditionals

Conditionals work similar to an `if then else` statement. They are defined like `(?(if)then|else)`

`(?(A)B|c)` this means if `A` then `B` else `c`

Conditionals can also be used to check if a captured group exists.

`(?(1)hello|hi)` this will check if the numbered group exists  
`(?(<mygroup>)hello|hi)` this will check if the named group mygroup exists

# More...

There's still more to learn with regex's like `recursion` and `subroutines` but I'm going to leave this post for now, maybe I'll update it later and I'll finish it off with some useful links.

# Useful links

Great resource with a lot of information, good for getting started with regular expressions.  
[Regular-expressions.info](http://www.regular-expressions.info/)

Onigmo is the engine used by Ruby since version 2.0.  
[Onigmo](https://github.com/k-takata/Onigmo)  
[Onigmo documentation](https://github.com/k-takata/Onigmo/blob/master/doc/RE)

Ruby's documentation on regular expressions.  
[Ruby-docs.org Regular Expressions](http://ruby-doc.org/core-2.4.1/doc/regexp_rdoc.html)  
[Ruby-docs.org Regexp Class](http://ruby-doc.org/core-2.4.1/Regexp.html)

A comparison of regular expression engines.  
[regular expression engines](https://en.wikipedia.org/wiki/Comparison_of_regular_expression_engines)

If you want to practice your regex skills you can play  
[regex golf](https://alf.nu/RegexGolf)  
[regex crosswords](https://regexcrossword.com/)
