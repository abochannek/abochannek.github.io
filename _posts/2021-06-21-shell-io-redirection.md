---
layout: post
title:  "Shell I/O Redirection (Part 2)"
date:   2021-06-21 21:15:10 -0700
last_modified: 2021-06-30 12:34:54 -0700
categories: Shell
related: [
	"Shell I/O Redirection (Part 1)"
	]

---

# Duplicating File Descriptors

In the previous post, the idea of duplicating or copying file
descriptors was introduced to redirect _both_ standard output and
standard error to the _same_ file.

When a command is executes interactively from the shell, the defaults
for `stdin`, `stdout`, and `stderr` are the controlling terminal.

![cmd standard I/O](/images/fdcopy.dot.2.png)

The goal, as stated above, is to redirect both `stdin` and `stderr` to
the same `file`.

![cmd redirect stdout stderr](/images/fdcopy.dot.3.png)

Redirecting `stdout` to `file` is easily achieved by `cmd >file`. But
what about `stderr` (file descriptor 2)? Simply adding a `2>file` or
even a `2>>file` to the previous command produces unpredictable
results: depending on the order in which the command sends its output
to `stdout` and `stderr`, one or the other or both may be recorded in
`file`.

The answer to this problem is to duplicate (or copy) an _existing_
file descriptor and the syntax in the POSIX-compatible shells is this:

`cmd 2>&1`

What this means is that `stderr` (file descriptor 2) is made to be a
copy of `stdout` (file descriptor 1.) In other words, `stderr` points
to where `stdout` points to.

This now suggests two possible approaches to redirect both `stdout`
and `stderr` to the same `file`:

- `cmd 2>&1 >file`
- `cmd >file 2>&1`

However, one of them is almost always _not_ what is intended.

#### Standard Output Duplication Before Redirection

Even though `stdout` and `stderr` point to the same terminal, let's
consider it two instances of the same device for demonstration
purposes.

![cmd redirect stdout stderr terminal](/images/fdcopy.dot.4.png)

In the first command, `2>&1` means that `stderr` now points to where
`stdout` pointed to.

![cmd copy stdout to stderr](/images/fdcopy.dot.5.png)

Now `stdout` is redirected (`>file`).

![cmd redirect stdout to file](/images/fdcopy.dot.6.png)

In other words:

- `stderr` continues to point to the terminal
- `stdout` points to `file`

#### Standard Output Duplication After Redirection

Starting from the same initial setup

![cmd redirect stdout stderr terminal](/images/fdcopy.dot.7.png)

first `stdout` is redirected (`>file`).

![cmd redirect stdout to file](/images/fdcopy.dot.8.png)

And _then_ `stderr` is pointed to where `stdout` points at (`2>&1`)

![cmd copy stdout to stderr](/images/fdcopy.dot.9.png)

This is likely what was intended and shows why the order of the
redirections matters.

#### Syntax for Duplicating File Descriptors

In these examples, `n` stands for an optional file descriptor
number. Just like for regular redirection, it is not necessary to use
0 for input and 1 for output redirection. `word` stands for a numeric
file descriptor, the character `-` to close a file descriptor
(discussed in a future post), or variables names for some shells as
mentioned in the previous post.

- Duplicate an input file descriptor: `n<&word`
- Duplicate an output file descriptor: `n>&word`

A process has access to at least up to 20 file descriptors to identify
open files. The POSIX-compatible shells can use the special command
`exec` to create file descriptors and for the input example below,
assume that a file descriptor number 3 already exists that points to a
file. Explicitly listing file descriptor 0 as an input is of course
unnecessary in this somewhat contrived example.

The output example is again using duplicating `stdout` for `stderr` to
and other file descriptors are possible here as well.

|       | Duplicate input file descriptor | Duplicate output file descriptor |
| :-    | :-                              | :-                               |
| POSIX | `cmd 0<&3`                      | `cmd 2>&1`                       |
| bash  | `cmd 0<&3`                      | `cmd 2>&1`                       |
| ksh   | `cmd 0<&3 `                     | `cmd 2>&1`                       |
| csh   | n/a                             | n/a                              |
| zsh   | `cmd 0<&3 `                     | `cmd 2>&1`                       |

> ### Other Shells
>
> The syntax for I/O redirection in POSIX-standard shells is arguably
> confusing and inconsistent. In the 10th Edition of Research Unix and
> the Plan 9 operating system developed at Bell Labs, Tom Duff created
> a new shell called `rc` that cleaned up much of the Bourne Shell's
> syntax. I/O redirection in particular is more consistent. Instead of
> `cmd >file 2>&1`, the `rc` shell uses `cmd >file >[2=1]`.
>
> Other Unix shells exist that do not attempt to be POSIX-compliant,
> but I/O redirection is rarely different.

Because making `stderr` a copy of `stdout` and then redirecting it to
a file is so common an operation, many shells have shortcuts for it.

|       | Redirect standard output and standard error |
| :-    | :-                                          |
| POSIX | `cmd >file 2>&1`                            |
| bash  | `cmd &>file` (preferred) or `cmd >&file`    |
| ksh   | `cmd >file 2>&1`                            |
| csh   | `cmd >&file`                                |
| zsh   | `cmd &>file` (preferred) or `cmd >&file`    |

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
