---
layout: post
title:  "Shell I/O Redirection (Part 3)"
date:   2021-07-06 20:39:53 -0700
categories: Shell
related: [
	"Shell I/O Redirection (Part 1)",
	"Shell I/O Redirection (Part 2)"
	]

---

This is the final installment in the _Shell Redirection_ series of
posts. It continues the discussion of file descriptor duplication and
then concludes with three topics related to I/O redirection: standard
error in pipelines, here-documents, and process substitution.

# `exec` and File Descriptor Variables

In the previous posts, I/O redirection was explained in terms of
redirecting input or output for a command. When the shell executes a
command, it first makes a new process (`fork`) and sets up the
redirections, before then executing (`exec`) the required
command. With the termination of the command, the forked process ends.

The shell builtin `exec` can be used to replace the running shell
process with another command: `exec date` would replace the running
shell with the `date` program, print the current date, and then
terminate.

However, if there is _no_ command specified, `exec` is used to control
file descriptors for the current shell process. Examples from the
POSIX standard:

* Open `readfile` as file descriptor 3 for reading: `exec 3< readfile`
* Open `writefile` as file descriptor 4 for writing: `exec 4> writefile`

Because of csh's limited redirection operators, this feature is only
available in the other, POSIX-compliant shells.

|       | Command to redirect I/O in the current shell |
| :-    | :-                                           |
| POSIX | `exec`                                       |
| bash  | `exec`                                       |
| ksh   | `exec`                                       |
| csh   | n/a                                          |
| zsh   | `exec`                                       |

As hinted at in [Part 1](../../06/12/shell-io-redirection.html#variables)
of this series of posts, non-numeric file descriptors are supported in
ksh93, Bash, and Zsh. Using the above `readfile` and `writefile`
examples, variables could be used like this:

```
exec {READ}<readfile
exec {WRITE}>writefile

cat <&$READ
ls >&$WRITE
```

When variables are used for redirection, the file descriptor number is
usually 10 or greater.

To experiment with creating additional file descriptors in Bash, the
following command can be used to observe the effect of an `exec`
command. It shows which file descriptors the shell currently has open,
whether they are opened for reading, writing, or both, and what file
or device the file descriptors point at. (The `lsof` program is often
already installed on Linux systems, but needs to be installed on macOS
for this.)

```
lsof -p $$ -Fan0 2>/dev/null | while read -r line
do if [[ $line =~ ^f([[:digit:]]+).*a(.*)n(.*)$ ]]
   then printf "fd %s (mode %s) ->%s\n"	${BASH_REMATCH[1]} ${BASH_REMATCH[2]} ${BASH_REMATCH[3]}
   fi
done
```

# Closing And Moving File Descriptors

When the additional file descriptors are no longer necessary, it is a
good practice to clean them up by closing them. The standard syntax
for this in the shells discussed (again, except csh) is:

`<&-`

The command to close the file descriptors 3 and `WRITE` from the
examples above is

`exec 3<&- {WRITE}>&-`

The use of `<` vs `>` does not matter except for when no file
descriptor is specific, in which case `stdin` and `stdout`,
respectively, are implied.

A convenient shorthand for dropping output, is to close `stdout` (or
`stderr`) like so: `cmd >&-`. Depending on the command, this may
result in an error though if the command can not handle a closed
`stdout`.

The uses of `{}` is required for variables on the close operator or
otherwise the syntax would be ambiguous because file descriptor
numbers 10 or greater are assigned to the variables.

Because an error in this command can lead to the shell exiting, ksh93
offers an alias called `redirect` that will prevent this from
happening even if file descriptors are mis-specified.

In some cases it may be convenient to duplicate and close a file
descriptor, i.e., to _move_ it. In this example, file descriptor 5 is
pointing where 3 is pointing at and 3 is closed.

|       | Close file descriptor | Move file descriptor |
| :-    | :-                    | :-                   |
| POSIX | `exec 3<&-`           | n/a                  |
| bash  | `exec 3<&-`           | `exec 5<&3-`         |
| ksh   | `exec 3<&-`           | `exec 5<&3-`         |
| csh   | n/a                   | n/a                  |
| zsh   | `exec 3<&-`           | n/a                  |


# `ksh93` Extended I/O

When opening a file for reading with I/O redirection, it is not
possible with shell features alone to move backwards in the file. In
other words, once a file has been read, it would need to be closed and
reopened with `exec` to start from the beginning.

Since the 1993 version of ksh, an interesting feature was added to I/O
redirection the resolves this.

| `<#((expr))`, `>#((expr))` | Evaluate the arithmetic expression `expr` and move to that byte position in `stdin` and `stdout`, respectively |
| `<#pattern`                | Seek `stdin` for the next occurrence of `pattern`                                                              |
| `<##pattern`               | Seek `stdin` for the next occurrence of `pattern` and send the skipped content to `stdout`                     |

Each of these expressions takes file descriptors other than the
default `stdin` and `stdout` and are used standalone, meaning an
`exec` command is not required.

# Pipelines and Standard Error

Pipeline were mention in the very beginning of this series of posts as
enabling the "Unix philosophy." All shells discussed here support
connecting the output of one command to the input of another through a
pipeline:

`cmd1 | cmd2`

Because it is also quite common to want to merge `stdout` and `stderr`
in a pipeline, a shortcut for it exists.

|       | Standard output and error in a pipeline |
| :-    | :-                                      |
| POSIX | n/a                                     |
| bash  | `cmd1 |& cmd2`                          |
| ksh   | _See note_                              |
| csh   | `cmd1 |& cmd2`                          |
| zsh   | `cmd1 |& cmd2`                          |

The csh syntax mirrors the `>&` construct for redirecting `stdout` and
`stderr`.

Ksh93 uses the operator `|&` for a different operation related to
co-processes and concurrency. Because `|&` is not a standardized
operator, it should not be used in shell scripts.

# Process Substitution

Process substitution is a powerful feature to execute commands and use
their input or output where another command may expect a
filename. It's similar to and somewhat more flexible than pipelines.

|       | Process output | Process input  |
| :-    | :-             |                |
| POSIX | n/a            | n/a            |
| bash  | `cmd1 <(cmd2)` | `cmd1 >(cmd2)` |
| ksh   | `cmd1 <(cmd2)` | `cmd1 >(cmd2)` |
| csh   | n/a            | n/a            |
| zsh   | `cmd1 <(cmd2)` | `cmd1 >(cmd2)` |

In practical terms, this means that if a command takes filenames as
arguments, process substitution can be used to reference the output of
another command without having to write to temporary files. Because
most commands can also take `stdin` in lieu of a filename, this
approach is especially useful when the output of multiple commands is
to be processed.

In this example, the current directory is listed (`ls`) and the lines
are counted (`wc -l`). This is done as a _command substitution_ as a
parameter to `seq` to generate a list of numbers. This list of numbers
is then used by `paste` to combine with the `ls` output:

`paste <(seq 1 $(ls|wc -l) ) <(ls)`

The input process substitution is used less commonly because commands
don't usually expect a filename argument for output -- I/O redirection
can be used for that. However, the `tee` command is an example of a
command that can send its input to one or more output files.

# Here Documents

It can be very convenient to embed data for commands directly in shell
scripts instead of reading them in from a file. That is what
here-documents are used for. A shorter form called a here-string is
also available.

|       | Here Doc | Here String |
| :-    | :-       |             |
| POSIX | `<<`     | n/a         |
| bash  | `<<`     | `<<<`       |
| ksh   | `<<`     | `<<<`       |
| csh   | `<<`     | n/a         |
| zsh   | `<<`     | `<<<`       |

A here-string is simply passing a short text string to `stdin` of a
command. The string itself is subject to expansion (e.g., parameters
or arithmetic expansion) before being passed to the command.

The following two constructs are equivalent:

`cmd <<< word` and `echo word | cmd`

A here-document allows for multi-line input and has a slightly more
complicated syntax:

```
cmd << EOF
here-document
EOF
```
The text between the two delimiters (`EOF` is often used by
convention) is sent to `stdin` of `cmd`. The text itself is also
subject to expansion _unless_ all or part of the _first_ delimiter is
quoted.

For example, the variable `HOME` contains the user's home directory.

```
cat << EOF
$HOME
EOF
```

results in the value of `HOME`, e.g., `/home/user1` to be output.

```
cat << "EOF"
$HOME
EOF
```

will simply print `$HOME`.

Finally, the operator `<<-` will result in `TAB` characters (and only
`TAB`, not spaces) to be removed from the here-document. This supports
better formatting in shell scripts.

# End

This third part concludes the _Shell I/O Redirection_ series of
posts. The simple redirection commands are easy to master and widely
known. Some of the slightly more complicated examples have a syntax
that is easy to forget or to misapply. Extensive file descriptor
manipulation is rarely used outside shell scripts, but at least a
basic understanding of it can help improve shell user's effectiveness.
