---
layout: post
title:  "Bash Pattern Matching (Part 3)"
date:   2021-10-30 19:26:45 -0700
last_modified:
categories: Bash Globbing
related: [
	"Bash Pattern Matching (Part 1)",
	"Bash Pattern Matching (Part 2)"
	]

---

In this last part of the _Bash Pattern Matching_ series, let's look at
shell options that affect matching as well as brace and tilde
expansion, which are often used alongside pattern matching.

# Filename Match Failures

When a pattern does not match any filenames, the pattern is simply
returned unchanged. Setting the `failglob` Bash option instead causes
the shell to display an error message.

In this example, no files in the directory exist that end with
`.txt`. The `ls` command will fail when `failglob` is not set. Upon
setting this option, the shell itself will give the error message and
`ls` will not be executed:
```
$ ls *.txt
ls: *.txt: No such file or directory
$ shopt -s failglob
$ ls *.txt
-bash: no match: *.txt
```

The related Bash option `nullglob` will simply replace a non-matching
pattern with an empty word instead of using the pattern literally or
failing (the `failglob` setting takes precedence over the `nullglob`
setting.)

A situation in which this is useful is when looping over a set of
patterns, but can produce unexpected results when there are no
matching patterns and the command behaves differently in that case:
`ls` will simply list _all_ files when invoked without arguments,
which is likely not what is intended.

Setting `failglob` and `nullglob` helps with script debugging and
should be used carefully in production. The `failglob` option is also
particularly useful for interactive shell use, but may interact in
unexpected ways with programmable completion.

# Controlling Which Files to Match

By default, globbing will not match the so-called hidden files, i.e.,
files with a name starting with a `.`. The Bash option `dotglob`
reverses this behavior; the special files `.` and `..` always require
explicit matching though.

The Bash variable `GLOBIGNORE` is a colon-separated patterns list that
can be used to specify what filenames _not_ to match during
expansion. The colon-separated list format is well known from the
shell `PATH` variable.

For example, to simply never complete filenames that end with `.jpg`,
`.gif`, or `.txt`, set `GLOBIGNORE="*.jpg:*.gif:*.txt"`. Refer to the
[Extended Pattern
Examples](../../09/10/bash-pattern-matching.html#extended-pattern-examples)
in the previous post for how to use extended patterns to accomplish
the same thing for one command.

# Extended Patterns, Ranges, and Case

Extended patterns were the subject of [Part
2](../../09/10/bash-pattern-matching.html) of this series of
posts. The `extglob` Bash option needs to be set to enable those
patterns.

Locales and collation -- the sort order of characters -- were
mentioned there too. The `globasciiranges` Bash option can be set to
override the locale-specific collation and instead use the standard
`C` locale for character range expressions. That means numbers, then
uppercase letters 'A' through 'Z,' and then lowercase letters 'a'
through 'z'.

Related to collation are the `nocaseglob` and `nocasematch` Bash
options. `nocaseglob` matches case-insensitively for filenames; `case`
and the Bash `[[` test statement follow the `nocasematch`
option. Case-insensitive pattern matching is not available for
parameter expansion, however.

As far as `nocaseglob` is concerned, not all file systems are
case-sensitive, which means that differently cased literal characters
are matched even if patterns are not. The best known example of this
is the macOS APFS file system in its default setting.

# Recursive Matching

Bash introduced recursive filename expansion with Bash 4.0
in 2009. Using a `**` in a pattern expands to all files, directories,
and subdirectories that match the pattern. It is enabled with the Bash
option `globstar`. Using the pattern `**/` will only expand to
directories and subdirectories. In later versions of Bash, symbolic
links are no longer traversed when doing recursive filename
expansion.

Recursive filename matching originated in Zsh in 1990 with its
earliest public release and the `**` syntax followed in
about 1992. Zsh has extensive functionality around what it calls
"Filename Generation" which can replace the need for a `find`
command. David Korn added the `**` syntax to ksh93 in 2003 and
apparently was unaware of its existence in Zsh, which is why there are
subtle differences in how they operate. The Bash functionality is more
similar to ksh93 than Zsh. For example, Zsh additionally offers `***`
to follow symbolic links.

When using `**`, it is worth considering how deep the search might go
because it can take quite a while to traverse all the
subdirectories. A more targeted `find` may be a better choice in some
cases.

# Brace Expansion

Brace expansion is a mechanism to construct words before any other
expansion takes place. It is purely textual and because of that is
frequently used to construct filenames for files that do not exist
yet. For example: `cp document{.txt,.old}` generates the line `cp
document.txt document.old`. There can be multiple strings within the
braces separated by commas as well as further brace expressions:

```
$ echo test{file_{1,2},data_{a,b}}
testfile_1 testfile_2 testdata_a testdata_b
```

As is common in the shell, to include the special characters
`{`, `}`, and `,` literally, they need to be escaped with a backslash
`\`.

With Bash 3.0 in 2004, sequence expressions were added to braces and
that simplifies constructing a series of files with similar names:

```
$ echo testfile{1..10}
testfile1 testfile2 testfile3 testfile4 testfile5 testfile6 testfile7
testfile8 testfile9 testfile10
```

Numbers can be prefixed with zeros to indicate a fixed number of
digits. In the above example `01..10` would create filenames from
`01`, `02`, ... to `09`, `10`.

Single letters can also be used and it is important to keep the
collating sequence in mind. Uppercase letters 'A' through 'Z' come
before lowercase letters 'a' through 'z' in ASCII. There are six
additional non-alphanumeric characters between `Z` and `a`, some of
which are special to the shell (like the backslash for example), so it
is strongly recommended _not_ to cross the boundary between upper- and
lowercase letters in ones sequence.

Depending on the start and end of the sequence, it will either count
up or down by one. An increment can also be specified:

```
$ echo {0..10..2}
0 2 4 6 8 10
```

# Tile Expansion

The use of the `~` for a user's home directory originated in the C
Shell in the Second Berkeley Software Distribution in 1979. An
unconfirmed conjecture states that `~` was chosen as the character for
this function by Bill Joy, because the ADM-3 terminal that was in use
at Berkeley had the `~` key also labeled as `HOME`, just like the
cursor keys were on the letter keys `H`, `J`, `K`, and `L`, which were
used for cursor movement in the vi editor.

The tilde without a username is the current user's home
directory. With a username, it is that user's home directory:
`~fred/foo` is `foo` in user `fred`'s home directory.

Tilde expansion also uses the directory stack; `~-` is the most
recent, `~+` the current directory. Numeric references to directories
on the stack are possible as well.

# Readline

The Readline library used by Bash for command editing and completion
has a number of function to interactively expand patterns. Many of
them are bound to key sequences by default. The `bind -P` command
lists all functions and bindings for the current session (including
unbound functions.)

Usually the `TAB` key will complete filenames, usernames, commands,
variables, etc. If there are multiple completion options, a second
`TAB` shows the possible completions. Individual functions for
expansion are also available, like `M-~` for tilde expansion (`M-` in
this context is the `Meta` key, which can either be the `Alt` or `Esc`
key.)

In the context of pattern matching for globbing, a couple of useful
functions are `glob-expand-word` (`C-x *`) and `glob-list-expansions`
(`C-x g`). The latter is effectively the same as hitting `TAB`
twice. The `glob-expand-word` function executes the filename expansion
and inserts the result right in the active command line.

# End

This concludes the _Bash Pattern Matching_ series of posts. The basic
patterns from the first part and some of the extended patterns from
the second part are commonly and regularly used on the command
line. The Bash options that control pattern matching are frequently
set in a Bash start-up file once they are found to be useful.

It is useful to familiarize oneself with the Readline functions that
make using Bash more productive. Additionally, Zsh has even more
extensive filename generation functionality that is worth exploring.
