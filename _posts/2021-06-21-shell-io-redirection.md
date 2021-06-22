---
layout: post
title:  "Shell I/O Redirection (Part 2)"
date:   2021-06-21 21:13:52
categories: Shell
---

# Duplicating File Descriptors

In the previous post, the idea of duplicating or copying file
descriptors was introduced with the example of redirecting _both_
standard output and standard error to the _same_ file.

When a command is executes interactively from the shell, the defaults
for `stdin`, `stdout`, and `stderr` are the controlling terminal.

![cmd standard I/O](/images/fdcopy.dot.png)

The goal, as stated above, is to redirect both `stdin` and `stderr` to
the same `file`.

![cmd redirect stdout stderr](/images/fdcopy.dot.2.png)

Redirecting `stdout` to `file` is easily achieved by `cmd >file`. But
what about `stderr` (file descriptor 2)? Simply adding a `2>file` or
even a `2>>file` to the previous command produces unpredictable
results.

The answer to this problem is to duplicate (or copy) an _existing_
file descriptor and the syntax in the POSIX-compatible shells is this:

`cmd 2>&1`

What this means is that `stderr` (file descriptor 2) is made to be a
copy of `stdout` (file descriptor 1.) In other words, `stderr` points
to where `stdout` points to.

This now suggests two possible approaches to redirect both `stdout`
and `stderr` to the same `file`. However, one of them is almost always
_not_ what is intended:

- `cmd 2>&1 >file`
- `cmd >file 2>&1`

#### Standard Output Duplication Before Redirection

Even though `stdout` and `stderr` point to the same terminal, let's
consider it two instances of the same device for demonstration
purposes.

![cmd redirect stdout stderr terminal](/images/fdcopy.dot.3.png)

In the first command, `2>&1` means that `stderr` now points to where
`stdout` pointed to.

![cmd copy stdout to stderr](/images/fdcopy.dot.4.png)

Now `stdout` is redirected (`>file`).

![cmd redirect stdout to file](/images/fdcopy.dot.5.png)

In other words:

- `stderr` continues to point to the terminal
- `stdout` points to `file`

#### Standard Output Duplication After Redirection

Starting from the same initial setup

![cmd redirect stdout stderr terminal](/images/fdcopy.dot.6.png)

first `stdout` is redirected (`>file`).

![cmd redirect stdout to file](/images/fdcopy.dot.7.png)

And _then_ `stderr` is pointed to where `stdout` points at (`2>&1`)

![cmd copy stdout to stderr](/images/fdcopy.dot.8.png)

This is likely what was intended and shows why the order of the
redirections matters.

#### Syntax for Duplicating File Descriptors

Not all shells discussed here have a general syntax for duplicating
file descriptors.

Using the syntax from the POSIX standard, `[n]` stands for an optional
file descriptor number. Just like for regular redirection, it is not
necessary to use `0` for input and `1` for output redirection.

In the below list `word` stands for a numeric file descriptor, the
character `-` to close a file descriptor (discussed in a future post),
or variables names for some shells as mentioned in the previous post.

- Duplicate an input file descriptor: `[n]<&word`
- Duplicate an output file descriptor: `[n]>&word`

For example purposes, file descriptors 3 and 4 are used as inputs and
5 and 6 as outputs.

|       | Duplicate input file descriptor | Duplicate output file descriptor |
| :-    | :-                              | :-                               |
| POSIX | `cmd 4<&3`                      | `cmd 6>&5`                       |
| bash  | `cmd 4<&3`                      | `cmd 6>&5`                       |
| ksh   | `cmd 4<&3 `                     | `cmd 6>&5`                       |
| csh   | n/a                             | n/a                              |
| zsh   | `cmd 4<&3 `                     | `cmd 6>&5`                       |
| rc    | `cmd <[4=3]`                    | `cmd >[6=5] file`                |

Because making `stderr` a copy of `stdout` and then redirecting it to
a file is so common an operation, many shells have shortcuts for it.

|       | Redirect standard output and standard error |
| :-    | :-                                          |
| POSIX | `cmd >file 2>&1`                            |
| bash  | `cmd &>file` (preferred) or `cmd >&file`    |
| ksh   | `cmd >file 2>&1`                            |
| csh   | `cmd >&file`                                |
| zsh   | `cmd &>file` (preferred) or `cmd >&file`    |
| rc    | `cmd >file >[2=1]`                          |

It is noteworthy that the C Shell does not have the generic file
descriptor duplication of the POSIX-compatible shells. The C Shell
shortcut for redirecting both `stdout` and `stderr` does exist in Bash
and Zsh, but the "reverse" syntax of `&>` is preferred. This is
because of ambiguities that can arise if the redirection target
expands to, e.g., the character `-`.

Finally, ksh93 and Zsh support a concurrent programming construct
called co-processes. To redirect from or to the co-process, those
shell use `>&p`. Bash also supports co-processes and uses variable
names as redirection sources and targets.

# Coming Up...

The next post will discuss closing and moving file descriptors and a
shortcut for redirections in pipelines. To wrap up file descriptor
copying, the `exec` special command will be explored as well.
