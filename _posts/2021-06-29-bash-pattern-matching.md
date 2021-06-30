---
layout: post
title:  "Bash Pattern Matching (Part 1)"
date:   2021-06-29 18:39:15 -0700
last_modified: 2021-06-30 13:04:23 -0700
categories: Bash Globbing
related: 
---

# Filename Expansion or "Globbing"

When working on the command line, very commonly a user wants to
specify a number of files whose names match a certain pattern: all
filenames starting with `proj`, ending with `.txt`, or files with a
three character filename for example. This can be achieved with
special wildcard characters to create filename patterns that the shell
then expands. First Edition Unix (_1971_) already offered that
capability to users and because it used a helper program called `glob`
(for "global") to expand the pattern, this mechanism is still called
"globbing."

The most common special characters to define filename patters are `*`
for zero or any characters and `?` for exactly one character. While
these look and act similar to regular expression quantifiers, it is
important to remember that they are different; in a regular expression
`.` is a single character, and `?`  matches a sequence of zero or one
of the preceding character.

Let's revisit the three examples above and see how `*` and `?` can be
used:

| Filenames starting with `proj` | `proj*` |
| Filenames ending with `.txt`   | `*.txt` |
| Three character filenames      | `???`   |

Before Unix, Multics (_ca. 1969_) used the `*` for filename patterns
and called this the "Star Convention." A pattern to match files was
called a "starname."  The similarities between the two systems and
regular expressions are not coincidental. Ken Thompson pioneered
regular expression use for pattern matching in editors. Together with
Dennis Ritchie he worked on Multics before Bell Labs withdrew from the
project. Thompson then developed Unix at Bell Labs together with
Ritchie who wrote the `glob` program.

The third pattern that was added to Unix globbing early on is the
bracket expression `[...]`. Any character listed between the brackets
is matched, so to match `a` or `b`, `[ab]` is used to match one of
them.

Bracket expressions also support ranges like `[a-z]` and
negations. The POSIX standard highlights that negation is achieved
with a `!` in the bracket (`[!ab]` means neither `a` nor `b`) as
opposed to the `^` used in regular expressions. The KornShell, which
the POSIX standard is based on, also uses `!`, whereas tcsh uses
`^`. Bash and Zsh allow either.

| Basic Filename Patterns |                                                 |
| :-                      | :-                                              |
| `?`                     | Match any single character                      |
| `*`                     | Match multiple characters including none        |
| `[...]`                 | Match a single character listed in the brackets |

In cases where the special characters are meant to be literal, they
need to be escaped. The easiest way to escape a single character is
with a backslash `\`. For example, to match the filename `how?` but
not `howl` use `how\?`.

# Pattern Matching Rules For Filename Expansion

While pattern matching _per se_ is just about strings, the use of it
for filename expansion is subject to some rules:

- The `.` at the beginning of a filename needs to be matched
  explicitly

Filenames that begin with a `.` are considered "hidden" and a `*` will
not match them, but a `.*` will. Bash has the `dotglob` shell options
to override this default behavior, which will be discussed in the next
post.

- The `/` in a file path needs to be matched explicitly

While obvious, this is an important rule because otherwise `*` would
match not only all entries in the current directory, but also in all
other directories in the current directory. In other words, in a
directory structure `projects/new/project.1.txt`, an `ls pro*` will
show that there is a `new` directory in `projects`, but not also match
`new/` and show the file in `new`.

- Any component of a path (i.e., directory) that is used for expansion
  need to have search permissions
- Any directory in a path that contains filenames to be expanded needs
  read permissions
  
Using the example above, the command `ls projects/*/project.1.txt`
requires read permission on the `projects` directory and search
permission on both `projects` and `new`.

# Pattern Matching For Things Other Than Filenames

Patterns are useful not only for filenames and over time found their
way into several other shell features.

#### Case Statements

When the shell is used as a scripting language, patterns fit naturally
in conditional expressions. The Mashey Shell (Programmer's
Workbench/Unix, _ca. 1975_) extended the original Thompson Shell with
programmability in mind and used patterns for the `switch`
statement. The C Shell picked up that feature and when the Bourne
Shell introduced the equivalent `case` statement, it also allowed the
same kind of patterns that previously were used for filename
expansion.

Example:
```
case "$input" in
   [A-Za-z]*)  echo "Letter" ;;
   [0-9]*)     echo "Number" ;;
   *)          echo "Unknown" ;;
esac
```

#### Test Commands

Another application for pattern matching is in test commands for `if`,
`while`, or `until` constructs. Even though pattern matching for tests
was introduced in the 1988 version of KornShell, POSIX did _not_ adopt
this. Therefore, when writing POSIX-compatible (or traditional Bourne)
shell code using the `test` or `[` utility pattern matching is not
available.

The KornShell introduced the conditional expression `[[...]]` and used
`=` for pattern matching. In its 1993 version `=` was declared
obsolete and `==` was introduced for pattern matching (`!=` is the
negated version) and `=~` for _regular expression_ matching. Bash and
Zsh use the same pattern.

Example:

```
if [[ "$input" == [A-Za-z]* ]]; then
   echo "Letter"
fi
```

Only the right-hand side of the comparison can be a pattern, the
left-hand side is either a string or evaluates to a string.

> ### `if`, `test`, `[`, and `[[`
>
> The `if` construct executes a test command to determine what to
> execute next. A command's exit value of `0` has the logical value of
> `true`, anything else is `false`. The `test` command has many
> options for file, string, or numeric tests. The `[` command is
> nothing else but `test` that expects `]` to be its last argument. In
> Bash, those are both shell builtin commands and they usually also
> exist as standalone commands for Bourne Shell use.
>
> A typical scenario would be to test if a file exists like `if [ -e
> file ]; then ...` or to check the value of a variable like `if [ $value
> -gt 5 ]; then ...`.
>
> The `[[` construct that originated in KornShell is a shell
> conditional construct like `if`, not a command, and implements
> practically all the same tests that `test` does. It also supports
> pattern (and regular expression) matching and has more flexible
> Boolean operators.
>
> As a rule of thumb to minimize surprises:
> - Use `[[` and `==` when using Bash, Zsh, or KornShell
> - Use `test`, `[`, and `=` when only a POSIX-compliant shell can be
>   assumed
> - Don't use `-a` or `-o` Boolean operators for `test` or `[` because
>   POSIX only specifies them as an extension

#### Parameter Expansion

Parameter expansion is how the shell expands variables to their
values. The variable `HOME` contains the current user's home directory
and the value can be accessed by `$HOME` or `${HOME}`. The latter form
is necessary to avoid ambiguity when using the options below.

Common parameter expansion expressions are `${#paramater}` for the
length of the variable value or `${parameter:-word}` to replace
substitute `word` if `parameter` is not set or null.

The POSIX standard, based on ksh88, specifies substring removal
options based on patterns -- ksh93 expanded that functionality with
substring substitution and Bash and Zsh adopted it. The following
examples illustrate the different functions.

In these examples, the variable `file` is set to
`projects/new/project.1.txt`.

| Function               | Example                            | Result                       |
| :-                     | :-                                 | :-                           |
| Remove shortest suffix | `${file%.*}`                       | `projects/new/project.1`     |
| Remove longest suffix  | `${file%%.*}`                      | `projects/new/project`       |
| Remove shortest prefix | `${file#*/}`                       | `new/project.1.txt`          |
| Remove longest prefix  | `${file##*/}`                      | `project.1.txt`              |
| Replace string         | `${file/[0-9]/A}`                  | `projects/new/project.A.txt` |
| Replace all strings    | `${file//project/plan}`            | `plans/new/plan.1.txt`       |
| Replace beginning      | `${file/#projects\/new/plan\/old}` | `plan/old/project.1.txt`     |
| Replace end            | `${file/%txt/doc}`                 | `projects/new/project.1.doc` |

The second to last example shows how to escape characters with special
meaning.

#### Programmable Completion

Interactive command line editing and filename completion are common
features in shells -- something that was not available in the original
Bourne Shell. Hitting the `<TAB>` key for filename or command
completion is a useful feature that came to Unix shell with tcsh. Ken
Greer, then at CMU, implemented a filename completion feature based on
the 1970s TENEX operating system for the DEC PDP-10 computers, which
was integrated into the C-Shell in 1981. This contributed to the
popularity of tcsh as an interactive shell.

Programmable completion is a feature that allows command-specific
completion to, e.g., list all options for a given command. Bash, Zsh,
and tcsh all use pattern matching for this feature.

# Coming Up...

In the next post on Bash Pattern Matching, extended patterns and
character classes will be discussed as will a number of shell options
that determine how globbing works. Finally, brace and tilde expansion
will be touched on, which are used together with, but are functionally
unrelated to filename expansion.
