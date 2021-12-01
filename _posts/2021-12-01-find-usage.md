---
layout: post
title:  "Using the find Command Effectively (Part 2)"
date:   2021-11-30 18:41:58 -0800
categories: Utilities
related: [
	"Using the find Command Effectively (Part 1)"
	]
---

# Contents
- [Listing the Found Files](#listing-the-found-files)
- [Acting on the Found Files](#acting-on-the-found-files)
- [`xargs`](#xargs)
- [Links](#links)
- [Depth and Pruning](#depth-and-pruning)

# Listing the Found Files

As already discussed in [Using the find Command Effectively (Part 1)](../../07/16/find-usage.html),
the default, implicit action for `find` is `-print`. This primary
outputs the current pathname, one per line, and always evaluates to
_true_.

Because of the special meaning of whitespace in the shell to
distinguish words, it can be problematic to process the output of
`-print` if the pathnames include spaces or newlines. To work around
this issue, the GNU version of `find` (as initially suggested by Dan
Bernstein) offers the `-print0` primary that uses the ASCII character
`NUL` to separate pathnames. The accompanying `-0` option in
[`xargs`](#xargs) is frequently used with it.

Another common, non POSIX-standard extension to `find` is the `-ls`
action, which appeared in the mid-1980s in BSD-based Unix
variants. Its output is equivalent to `ls -dils`.

> ### Unix Standardization
>
> The operating systems of the Unix family have undergone many changes
> and attempts at standardization. POSIX is still considered a
> reliable base level of available features, even for the many Linux
> distributions.
>
> Reconciling the split between the AT&T Unix and Berkeley BSD
> variants of Unix proved difficult and many compromises had to be
> made. The `-ls` action in `find`, which itself is not part of POSIX,
> illustrates how complicated that effort was.
>
> When `find`'s `-ls` first appeared in BSD and SunOS, ca. 1986, the
> documentation mentioned it as equivalent to `ls -dgils`. BSD's `ls
> -l` did not list group ownership, which created the need for an
> additional flag, `-g`, to include it alongside the user
> owner. However, for the `ls` command as standardized in POSIX in
> 1992, the `-g` option implies a `-l`, but only shows group ownership
> and omits the user owner.
>
> In POSIX the `-g` flag for `ls` is marked as part of the X/Open
> Option, an earlier standardization effort based on AT&T Unix System
> V.

# Acting on the Found Files

A number of `find` implementations support actions that delete
(`-delete`) or back up (`-cpio`) files. The most common (and only
POSIX-compliant actions on files are) are `-exec` and `-ok`.

In the [previous post](../../07/16/find-usage.html) in this series, an
example was used from the early Unix manual pages to remove
files. Omitting the tests, the command using more current quoting
practices looks like this:

```
find / … -exec rm {} \;
```

This command would execute the `rm` command with each pathname that
matches the tests. The `{}` are placeholders for the pathname, so that
it does not need to be the last argument to the command to be
executed. The `;` terminates the `-exec` primary, quoting is necessary
for the shell not to mistake it as a command separator.

In the case of destructive operations like file removal, the `-ok`
primary may be a better choice; each command will be output to
standard error, followed by a question mark, and the command will be
executed if the user responds in the affirmative. In practice, that
means a `y`, at least in English language locales.

### `-exec ;` vs `-exec +`

The `-exec` primary can also be terminated by a `+`, which changes how
the command will be invoked. Sets of pathnames will be grouped in one
execution up to a system-defined limit, but no less than 4096
bytes. This means that instead of for each matching file, the command
will usually only be invoked once, which requires a lot less system
resources.

The `+` must immediately follow the `{}`, which somewhat limits the
command that can be used for that style of `-exec`. For example,
`-exec cp {} dir +` is not allowed! Furthermore, there is no `-ok {} +`.

This feature, which according to the POSIX standard originated in
AT&T's System V, attempts to address the same issue that `-print0`
also solves. By passing sets of pathnames to the command, spaces and
newlines in filenames do not break them up into multiple file as would
happen in a pipeline with `xargs`.

There is another difference between `-exec … {} ;` and `-exec … {} +`.
When the primary is terminated by a `;`, it's the exit code of the
command that determines whether `-exec` results in a _true_ or
_false_. When it is terminated with a `+`, it is always _true_ and the
exit code of `find` itself reflects if any of the `-exec` invocations
resulted in a non-zero exit code.

The following example in a directory with two files demonstrates this
behavior:
```
$ find . -type f \( -exec false {} \; -o -print \)
./test1
./test2
$ echo $?
0
$ find . -type f \( -exec false {} + -o -print \)
$ echo $?
1
```
In the first case, the `false` does not prevent the `-print` because
it uses a logical `OR`. The exit code for `find` is zero, because no
error occurred. In the second case, the `false` short-circuits the
`OR` (because it is _true_), but the exit code is non-zero.

# `xargs`

The `xargs` command -- just like `find` -- originated in the
Programmer's Workbench (PWB) version of Unix. It takes text from
standard input and uses it as arguments for a command. It is
frequently used on conjunction with `find` where it offers more
flexibility than the `-exec` action.

A simple example for `xargs` usage is to archive all files with a
specific filename to a `tar` archive.

```
find . -type f -name \*.bak -print0 | xargs -0 tar -cf ../backup.tar
```

This pipeline will first find all files (relative to the current
directory) that end in `.bak` and then pass that list of files to the
`tar` command to create the archive `backup.tar`. As mentioned above,
`-print0` is used to resolve issues with whitespace in filenames; the
`-0` counterpart in `xargs` expects the `NUL`-separated words to be
passed in. This particular example could have also been accomplished
using `-exec +`. The `-exec +` primary to `find` is in fact
recommended over `xargs` in the POSIX standard.

The `xargs` utility supports several useful options. The `-p` option
will prompt before command execution. This is similar to `find`'s
`-ok`, for which no `-ok +` exists.

The `-n` and `-L` options specify how many input words or lines,
respectively, should be used for each command invocation.

For example, the command `paste <( seq 1 10 ) <( seq 1 10 )` creates
two columns of the numbers one through ten.
```
1	1
2	2
3	3
4	4
5	5
6	6
7	7
8	8
9	9
10	10
```
This demonstrates the difference between `-n` and `-L`:
```
… | xargs -n 4
1 1 2 2
3 3 4 4
5 5 6 6
7 7 8 8
9 9 10 10
```
```
… | xargs -L 4
1 1 2 2 3 3 4 4
5 5 6 6 7 7 8 8
9 9 10 10
```

Finally, the `-I` option is used to specify the replacement string,
which often is chosen to be `{}` to mimic `find`, but can be pretty
much any string. By default, the replacement string can be used in up
to five arguments. A common use case is when the variable argument is
not the last argument to the command, which is not possible with `find
-exec +`.

`find . -type f -print0 | xargs -0 -I {} cp {} /tmp`

# Links

One of the challenges in using `find` is to determine whether or not
to follow symbolic links. By default, `find` will not follow _any_
symbolic links, whether they are on the command line or encountered
during search. The GNU version of `find` has the `-P` option to make
this behavior explicit.

The `-H` option causes `find` to follow symbolic links on the command
line only; the `-L` option will also make it follow symbolic links it
encounters during traversal. In both cases, should the link not point
to a valid file, the link itself will be used.

The following example demonstrates the different options. In a directory with these files:
```
file
link -> file
link_dir -> .
```
finding a regular file depends on the link option used.
```
$ find -P link_dir -type f
$ find -H link_dir -type f
link_dir/file
$ find -L link_dir -type f
link_dir/file
link_dir/link
```

Historically, the `-follow` primary was used for what the `-L` option
implements, but the latter is more consistent and `-follow` is
considered obsolete.

Because symbolic links can span file system boundaries, it is probably
advisable to use the `-xdev` primary to prevent this from happening
when the `-L` option is used. Another name for this option (from AT&T
Unix System V) was `-mount`. The File system type can be matched with
the non-standard `-fstype` and many `find` implementation also support
a `-local` to avoid traversing network-mounted file systems.

# Depth and Pruning

Pruning, i.e., ignoring directories during traversal, is a common
operation, which is somewhat complicated by `find`'s syntax. The basic
approach is this:

`find startdir \( -name dir1 -o -name dir1 … \) -prune -o …`

The `-name` primary can of course be any other test as well, `-path`
is often used to ignore patterns of directory paths.

Additionally, limiting the depth of the traversal is quite useful and
can be achieved by pruning paths or in the GNU version of `find` the
`-maxdepth` primary.

The following two commands are equivalent.
```
find . -path '*/*/*/*' -prune -o -print
find . -maxdepth 2
```

Confusingly, the unrelated `-depth` primary is _not_ used for limiting
`find`, but to instead do a depth-first search. This results in files
in directories being listed before the directories that contain
them. This feature was originally useful in conjunction with the
`cpio` backup tool in order to restore files in directories without
write permission. The GNU version of `find` also offers the `-d`
option instead of `-depth`.

# End

This concludes the two part series on _Using the find Command
Effectively_. Despite its somewhat unusual syntax, the `find` command
is very much worth learning. It is a useful tool to locate files,
identify changes to the system, and categorize groups of files based
on their properties. Together with `xargs` it can be used to construct
powerful pipelines. The Boolean operators that connect the primaries
are quite different in syntax from any other Unix command, but also at
the root of `find`'s power.
