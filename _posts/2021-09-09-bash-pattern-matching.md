---
layout: post
title:  "Bash Pattern Matching (Part 2)"
date:   2021-09-09 20:23:31 -0700
last_modified: 2021-10-09 20:12:47 -0700
categories: Bash Globbing
related: [
	"Bash Pattern Matching (Part 1)",
	"Bash Pattern Matching (Part 3)"
	]

---

# Extended Patterns

The pattern matching rules discussed in the first part of this post
don't offer a lot of flexibility. Only the occurrence of characters
can be matched, but not combinations of different patterns. That's
where _extended patterns_ come in, which are closer to regular
expressions. Bash introduced them with version 2.02 in 1998, where the
implementation followed ksh88 -- these patterns are not part of the
POSIX standard however.

Extended patterns first need to be explicitly enabled with the `shopt`
(Shell Options) builtin. Then check if the option is on:
```
$ shopt -s extglob
$ shopt extglob
extglob        	on
```
To keep `extglob` set for all sessions, it can be put into the user's
`.bashrc` in their home directory. It is important to check if that
file is read by the user's `.bash_profile` or `.profile` as is
customary or otherwise the `shopt` will not be executed for login
sessions.

Extended patterns are defined as a list of one or more patterns (which
themselves can be extended patterns) separated by a `|` and enclosed
in parentheses. The following five composites patterns are defined:

| `?(patterns)` | Match zero or one of the patterns    |
| `*(patterns)` | Match zero or more of the patterns   |
| `+(patterns)` | Match one or more of the patterns    |
| `@(patterns)` | Match exactly one of the patterns    |
| `!(patterns)` | Match anything _except_ the patterns |

The first three composite patterns use symbols based on regular
expressions quantifiers. The main difference again is the use of `?`
in globbing pattern matching, which stands for one and only one,
whereas in regular expressions and extended patterns it stands for
zero or one.

# Extended Pattern Examples

The easiest way to understand extended patterns is to work through
some examples.

Excluding filename patterns for globbing is likely the most common use
of extended patterns. For example, to list all files that do not end
with certain filename extensions, the `!(...)` is used:

`ls !(*.jpg|*.gif|*.txt)`

The maybe surprising aspect of this is the inclusion of the `*` inside
the parentheses. This is an artifact of the definition of this
pattern, which matches anything _except_ the patterns; another way to
look at this is that there is already an "implicit" `*` for matching
anything else.

The pattern list can include any basic (or extended) filename pattern
itself, which can simplify some patterns. To exclude a range of
characters, bracket expressions can be used:

`ls !([a-g]*)`

The other composites are best explained using a very simple pattern,
like one or two characters to match. Let's assume that text files for
a new project are labeled:

```
project-a.txt
project-aa.txt
project-b.txt
project-ab.txt
project-ba.txt
project-bb.txt
```
...and so forth. Matching a specific set of files is done using the
`@(...)`:

`project-@(a|b|ba).txt` results in
```
project-a.txt
project-b.txt
project-ba.txt
```

This particular match is not achievable using basic filename patterns
because of the different length of patterns.

In this example, `project-?(a|b|ba).txt` would have been given the
same result, because no file without the second filename component
existed. If however the same files had existed without the `.txt`
extension, a pattern to find all those files is:

`project-@(a|b|ba)?(.txt)`

How should the pattern look to match these files then?
```
project-ab
project-ab.txt
project-ab.txt.txt
...
```

| Pattern             | Matches                                                  |
| :-                  | :-                                                       |
| `project-ab.txt`    | `project-ab.txt`                                         |
| `project-ab?(.txt)` | `project-ab` and `project-ab.txt`                        |
| `project-ab+(.txt)` | `project-ab.txt` and `project-ab.txt.txt`                |
| `project-ab*(.txt)` | `project-ab`, `project-ab.txt`, and `project-ab.txt.txt` |

When used with pattern lists, the `+(...)` and `*(...)` patterns do
not limit the order or maximum number of pattern matches, which
simplifies matches significantly.

To match all files in the above list (`-a`, `-aa`, `-b`, `-ab`, etc.) a
simple `project-+(a|b).txt` is sufficient.

Finally, nesting extended patterns is possible, but can be slow if the
patterns get too complicated or match against long strings. Using the
above filenames again, the pattern to match `project-a.txt`,
`project-b.txt`, _and_ `project-bb.txt` is `project-@(a|+(b)).txt`.

# Character Classes

The term "character class" is generally used to refer to characters
(or character ranges) in brackets in either a shell pattern or a
regular expression. These types of character classes have been part of
Unix shell globbing since 1972, shortly after the introduction of `?`
and `*` as wildcard characters.

In the 1980s the importance of standards for internationalization of
software increased. AT&T offered the UNIX System V Release 4
Multi-National Language Supplement and when X/Open -- a business
consortium of originally European Unix vendors -- sought to
standardize internationalization, Hewlett Packard submitted its
National Language Support System. The result was the XPG2 standard,
which included internationalization features for Unix in 1987. Further
standardization followed for both the Unix operating system and the C
programming language. By 1992, the POSIX.2 standard included what is
now called POSIX Character Classes.

POSIX calls the expressions that are surrounded by brackets, bracket
expressions. A POSIX character class is a separate bracket notation
_within_ those brackets:

- `[A-Z]` is the bracket expression for the uppercase Latin characters
  A through Z
- `[[:upper:]]` is the bracket expression that contains the character
  class for all uppercase characters
  
The main difference is that the latter includes any uppercase
character regardless of whether they are in the A through Z
range, e.g., non-Latin uppercase characters. This is a significantly
more flexible approach and independent of the character set encoding
used.

The following character classes are supported in POSIX-compliant
systems and each _locale_ may support additional ones. A locale
defines language and cultural conventions for the user session to,
among other things, display localized text or date formats and define
the character encoding used.

| `[:alnum:]` | alphabetic and numeric | `[:alpha:]`  | alphabetic                     |
| `[:blank:]` | spaces and tabs        | `[:cntrl:]`  | control characters             |
| `[:digit:]` | digits                 | `[:graph:]`  | non-blank characters           |
| `[:lower:]` | lowercase alphabetic   | `[:print:]`  | blank and non-blank characters |
| `[:punct:]` | punctuation            | `[:space:]`  | all whitespace                 |
| `[:upper:]` | uppercase alphabetic   | `[:xdigit:]` | hexadecimal digit characters   |

Bash also supports two other, rarely used POSIX extensions:
equivalence classes and collating sequences. Their availability
depends on the whether or not they are defined in the locale used.

Equivalence classes allow the use of one character to stand in for
equivalent other characters. For example, if "n" and "ñ" are defined
to be equivalent, the bracket expression `[[=n=]]` matches either.

Collating sequences are primarily used when languages use more than
one glyph as a single logical character for collating purposes. For
example, in Welsh the sequences "ng", "ff", "ll", and several other
are considered single characters. If there is a collating symbol `ng`
defined, the bracket expression `[[.ng.]]` then matches "ng" as a
single character.

# Coming Up...

The remaining topics on Bash Pattern Matching will be discussed in a
final, third post on the topic: shell options that determine how
globbing works and brace and tilde expansion. As mention in the first
post, the latter two are functionally unrelated to filename expansion,
but are frequently used together.
