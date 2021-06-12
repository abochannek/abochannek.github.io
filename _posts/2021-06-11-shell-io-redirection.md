---
layout: post
title:  "Shell I/O Redirection (Part 1)"
date:   2021-06-11 21:37:16
categories: Shell
---

# Introduction

Redirecting input to and output from a command is one of the most
fundamental features of interacting with Unix (and by extension,
Linux) systems. The Thompson Shell in 1st Edition Unix (_1971_)
already implemented this concept and pipelines followed in 3rd Edition
(_1973_). Together, I/O redirection and pipelines enable the "Unix
philosophy" of small, single purpose tools that can be tied together
for more powerful applications.

Even though I/O redirection is so foundational to Unix, the syntax for
it can trip up even the most experienced users. The purpose of this
post is to highlight the syntactic and functional differences between
shells as they exist in 2021. Most of the shells discussed are in the
Bourne Shell family, with the exception of the C Shell and Rc. The C
Shell was developed before Bourne and Rc was a later simplification of
the Bourne Shell. The POSIX standard adopted the 1988 version of the
KornShell, which itself was based on the Bourne Shell.

The shells are:
* POSIX standard shell
* Bash
* KornShell (ksh93)
* C Shell (also applies to tcsh)
* Zsh
* Rc (10th Edition Unix and Plan 9)

This first part of a series of posts shows basic input and output
redirection, appending, and clobbering. Future posts discuss moving
file descriptors around, pipelines, and here-documents.

# Simple Input and Output Redirection

As reminder, the default file descriptor numbers a process uses are:

| `stdin`  | 0 |
| `stdout` | 1 |
| `stderr` | 2 |

While a numeric file descriptor can be used for practically all
redirection operators, in the case of simple `stdin` and `stdout`
redirection it can be omitted.

|           | Redirect standard output | Redirect standard input |
| :-        | :-                       | :-                      |
| POSIX     | `cmd > file`             | `cmd < file`            |
| bash      | `cmd > file`             | `cmd < file`            |
| ksh       | `cmd > file`             | `cmd < file`            |
| csh       | `cmd > file`             | `cmd < file`            |
| zsh       | `cmd > file`             | `cmd < file`            |
| rc        | `cmd > file`             | `cmd < file`            |

Redirecting a different file descriptor is demonstrated here by
redirecting `stderr` to a file. The syntax for input redirection is
analogous.

|           | Redirect standard error |
| :-        | :-                      |
| POSIX     | `cmd 2> file`           |
| bash      | `cmd 2> file`           |
| ksh       | `cmd 2> file`           |
| csh       | n/a                     |
| zsh       | `cmd 2> file`           |
| rc        | `cmd >[2] file`         |

Two points are important to make here:
* Rc uses a very different, and (as will become apparent in future
  posts) an arguably more rational syntax
* Csh and tcsh are more limited than Bourne-family shells and do not
  support specific file descriptors in their redirection syntax

A common workaround for Csh would be to use a sub-shell like so: `( cmd > /dev/tty ) >& file`

In addition to numeric file descriptors, ksh93, Bash, and Zsh also
allow variable names to be used for file descriptors that will be
assigned a file descriptor number greater than 10 and live beyond the
execution of the command.

This advanced technique is useful in shell scripts and a practical
example would look like this:

```shell
exec {newfd}>output
cmd >&$newfd
exec {newfd}>&-
```

Future posts will explain how to open, move, and close file
descriptors.

# Redirecting Input and Output

This feature is rarely used and is primarily useful when interacting
with bidirectional communication channels that are represented as
files (e.g., a remote host reachable under `/dev/tcp`.)

|           | Redirect both standard input and output |
| :-        | :-                                      |
| POSIX     | `cmd <> file`                           |
| bash      | `cmd <> file`                           |
| ksh       | `cmd <> file`                           |
| csh       | n/a                                     |
| zsh       | `cmd <> file`                           |
| rc        | `cmd <> file`                           |

# Appending and Clobbering

By default, output redirection creates a file if it doesn't exist and
overwrites an existing file. Overwriting may not be desirable and an
append operation can be used.

|           | Append standard output |
| :-        | :-                     |
| POSIX     | `cmd >> file`          |
| bash      | `cmd >> file`          |
| ksh       | `cmd >> file`          |
| csh       | `cmd >> file`          |
| zsh       | `cmd >> file`          |
| rc        | `cmd >> file`          |

Additionally, a number of shells (originating with csh) support a
`noclobber` variable that prevents overwriting unless a special syntax
is used. If that variable is set, a normal output redirection to an
existing file will fail.

|           | Redirect standard output and ignore `noclobber`|
| :-        | :-                                             |
| POSIX     | `cmd >| file`                                  |
| bash      | `cmd >| file`                                  |
| ksh       | `cmd >| file`                                  |
| csh       | `cmd >! file`                                  |
| zsh       | `cmd >| file` or `cmd >! file`                 |
| rc        | n/a                                            |

If `noclobber` prevents overwriting an existing file, what happens
when an append to a _non-existing_ file is attempted? Most shells
simply create the file, but Csh will fail unless a clobbering
redirection is used.

Zsh is a special case in that both a `NO_CLOBBER` and an
`APPEND_CREATE` option exist. File creation will fail if `NO_CLOBBER`
is set unless `APPEND_CREATE` is set (the POSIX-compliant approach) or
a clobbering redirection is used.

|           | Append standard output and ignore `noclobber` |
| :-        | :-                                            |
| POSIX     | n/a                                           |
| bash      | n/a                                           |
| ksh       | n/a                                           |
| csh       | `cmd >>! file`                                |
| zsh       | `cmd >>| file` or `cmd >>! file`              |
| rc        | n/a                                           |

# Coming Up...

The next post will dive into copying and moving file descriptors. The
most common case in which this is used is to redirect _both_ standard
output and standard error to the _same_ file. Here is a preview of
this operation.

|           | Redirect standard output and standard error |
| :-        | :-                                          |
| POSIX     | `cmd > file 2>&1`                           |
| bash      | `cmd &> file` (preferred) or `cmd >& file`  |
| ksh       | `cmd > file 2>&1`                           |
| csh       | `cmd >& file`                               |
| zsh       | `cmd &> file` (preferred) or `cmd >& file`  |
| rc        | `cmd > file >[2=1]`                         |
