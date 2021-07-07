---
layout: post
title:  "Shell I/O Redirection (Part 1)"
date:   2021-06-11 19:04:53 -0700
last_modified: 2021-07-06 17:52:56 -0700
categories: Shell
related: [
	"Shell I/O Redirection (Part 2)",
	"Shell I/O Redirection (Part 3)"
	]
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
shells as they exist in 2021. The shells discussed are in the Bourne
Shell family, with the exception of the C Shell. The C Shell was
developed before Bourne and many of its features were adopted by later
shells. The POSIX standard shell is based on the 1988 version of the
KornShell, which itself is compatible with the Bourne Shell and
incorporated many additional features of the C Shell.

The shells are:
* POSIX standard shell
* Bash
* KornShell (ksh93)
* C Shell (also applies to tcsh)
* Zsh

This first part of a series of posts shows basic input and output
redirection, appending, and clobbering. Future posts discuss moving
file descriptors around, pipelines, and here-documents.

# Simple Input and Output Redirection

File descriptors are numeric handles a running process uses to
identify open files. Unix systems also use the abstraction of a "file"
to communicate with I/O devices. The POSIX standard defines that a
process should have at least 20 different file descriptors available
to it at any given time. By convention the default file descriptor
numbers for I/O a process uses are:

![cmd standard I/O](/images/fdcopy.dot.png)

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

Csh and tcsh are more limited than Bourne-family shells and do not
support specific file descriptors in their redirection syntax. A
common workaround for Csh would be to use a sub-shell like so:
```( cmd > /dev/tty ) >& file```

> ### Variables
> 
> In addition to numeric file descriptors, ksh93, Bash, and Zsh also
> allow variable names to be used for file descriptors that will be
> assigned a file descriptor number greater than 10 and live beyond
> the execution of the command.

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

# Appending and Clobbering

By default, output redirection creates a file if it doesn't exist and
overwrites an existing file -- this is called "clobbering" the
existing file. Overwriting may not be desirable and an append
operation can be used.

|           | Append standard output |
| :-        | :-                     |
| POSIX     | `cmd >> file`          |
| bash      | `cmd >> file`          |
| ksh       | `cmd >> file`          |
| csh       | `cmd >> file`          |
| zsh       | `cmd >> file`          |

Additionally, a number of shells (originating with csh) support a
`noclobber` variable that prevents overwriting an existing file.

|       | Setting the `noclobber` shell variable |
| :-    | :-                                     |
| POSIX | `set -C` or `set -o noclobber`         |
| bash  | `set -C` or `set -o noclobber`         |
| ksh   | `set -C` or `set -o noclobber`         |
| csh   | `set noclobber`                        |
| zsh   | `setopt noclobber`                     |

If that variable is set, a normal output redirection to an existing
file will fail. There is however a redirection operator to ignore
`noclobber` for output redirection.

|           | Redirect standard output and ignore `noclobber`|
| :-        | :-                                             |
| POSIX     | `cmd >| file`                                  |
| bash      | `cmd >| file`                                  |
| ksh       | `cmd >| file`                                  |
| csh       | `cmd >! file`                                  |
| zsh       | `cmd >| file` or `cmd >! file`                 |

If `noclobber` prevents overwriting an existing file, what happens
when an append to a _non-existing_ file is attempted? Most shells
simply create the file, but Csh will fail unless a clobbering
redirection is used.

Zsh is a special case in that both a `CLOBBER` and an `APPEND_CREATE`
option exist (Zsh options are case insensitive and underscores are
ignored.) File creation will fail if `NO_CLOBBER` is set unless
`APPEND_CREATE` is set (the POSIX-compliant approach) or a clobbering
redirection is used.

|           | Append standard output and ignore `noclobber` |
| :-        | :-                                            |
| POSIX     | n/a                                           |
| bash      | n/a                                           |
| ksh       | n/a                                           |
| csh       | `cmd >>! file`                                |
| zsh       | `cmd >>| file` or `cmd >>! file`              |

# Coming Up...

The next post will dive into copying and moving file descriptors. The
most common case in which this is used is to redirect _both_ standard
output and standard error to the _same_ file.
