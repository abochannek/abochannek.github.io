---
layout: post
title:  "About the ps Command's Options"
date:   2021-12-29 01:44:04 -0800
categories: Utilities
---

# Contents
- [About the `ps` Command](#about-the-ps-command)
- [The Origin of `ps` Option Styles](#the-origin-of-ps-option-styles)
- [`ps` Option Categories](#ps-option-categories)
- [Which processes to show](#which-processes-to-show)
- [What to show about the processes](#what-to-show-about-the-processes)
- [How to format the output](#how-to-format-the-output)
- [Conclusion](#conclusion)

# About the `ps` Command

The `ps` command is the main administrative tool in Unix-family and
Linux systems to monitor processes. It has a vast number of options,
which historically have varied significantly across different systems,
even though the basic function of the command is the same. The primary
difference is that some versions of the `ps` command use the customary
`-` (dash or minus) to introduce options and some do not.

The origin of those differences lie in the 1980s split between AT&T's
commercial Unix (System III and System V) and Berkeley's BSD
distribution. Both were used by different Unix system vendors for
their commercial offerings, which resulted in confusing
inconsistencies for Unix system administrators who worked in mixed
environments -- a common situation in the 1980s and 1990s.

To quote the POSIX standard:

> There is very little commonality between BSD and System V
> implementations of ps. Many options conflict or have subtly
> different usages.

# The Origin of `ps` Option Styles

The `ps` (process status) command originally appeared in Unix 3rd
Edition alongside `kill` as super-user only commands. They were
documented in the maintenance command section (section 8) of the
manual dated January 20, 1973.

This very early version of `ps` had options and output that looked
quite different from later `ps` versions. In late-1973 4th Edition
Unix, `ps` moved to the command section (section 1) of the manual. It
had flags and produced output more in line with what a system
administrator today would recognize. By 1975, `ps` had headers in its
output and the supported flags had settled on `a`, `k`, `l`, and `x`,
which (with the exception of the `k` option) still have the same
meaning today.

While further editions of Research Unix did not document the use of
the dash for the options to `ps`, they were actually supported in the
code of 6th and 7th Edition and the convention is acknowledged
elsewhere in the manuals. Other AT&T Unix versions like the
Programmer's Workbench (PWB) or the AT&T-internal CB-Unix document the
use of a dash for the `ps` command.

> ### Flags vs. Options
>
> Unix command arguments that are preceded by a minus (dash) are
> usually referred to as "options." However, `ps` has tradtionally
> referred to them as "flags" and both of those terms are used in this
> post.
>
> The reason for indicating options with a special character is to
> differentiate them from filename arguments. This approach is based
> on the control-argument convention in the Multics operating system.
>
> In the earliest versions of Unix, there briefly existed a
> alternative convention to use a `+` for options. Before GNU settled
> on `--` for long options (_ca. 1991_), it used a `+` too and that
> was phased out by about 1993.

The earliest BSD version that included its own `ps` command was 3BSD
(_1979_). Prior to that, BSD was a collection of add-ons to AT&T's
Research Unix. The 7th Edition of Research Unix was ported to the DEC
VAX by a group of engineers at Bell Labs different from the Research
Center team; this version was called Unix/32V and was the basis of
3BSD. Because 32V did not implement paging virtual memory, 3BSD
reimplemented the memory management system, which necessitated some
significant changes to the `ps` command. It was 3BSD where the
commonly used option combination `aux` originated.

In 1981, AT&T announced a supported version of Unix that combined
Research Unix and several other, internal Unix variants as System
III. That version not only used dashes for `ps` flags, but also
changed the meaning of them. This is where the option idiom `-ef`
appeared. It's unclear which of the System III antecedents (if any)
changed the flag meaning. Because so little of CB-Unix survives and
the other Unixes that contributed to System III did not change the
flags' meanings, CB-Unix is a possible candidate for where this
evolution happened.

POSIX included `ps` in the POSIX.2a User Portability Extension
(_1992_) and limited the options to a small subset of what was in
common use across System V and BSD systems at the time. The
XSI-extensions specified in the POSIX standard are based on the X/Open
standard XPG2 (_1987_) which was largely identical to System V
Release 3. Because of its origins, the POSIX-standard `ps` uses dashes
and System V semantics for its flags.

The GNU implementation of `ps` used in Linux supports the GNU-typical
long options preceded by two dashes in addition to the Unix and BSD
styles.

# `ps` Option Categories

The `ps` command options roughly fall into these categories:

- Which processes to show
- What to show about the processes
- How to format the output

Depending on the specific implementation of `ps`, a small number of
options exist to, e.g., provide help or debug information. Especially
in older versions of the command, options existed to instruct `ps`
from which system image or memory file to read, which is used in
debugging system crashes.

The GNU version of `ps` is implemented using the Linux `/proc`
filesystem (_1992_), which offers a standardized interface to commands
that otherwise would need direct access to kernel memory. This
abstraction was original work in 8th Edition Research Unix (_1985_),
which itself was based on BSD 4.1c. AT&T's commercial System V
acquired `/proc` with the SVR4 release (_1989_), which was licensed
for numerous other commercial Unix versions, most notably Sun's
Solaris (_1992_).

# Which Processes to Show

The POSIX standard specifies that `ps` without any options displays
the running processes owned by the user invoking the command; only
processes associated with the terminal on which `ps` was invoked shall
be displayed. This is how GNU `ps` operates – macOS however does not,
despite its certification as UNIX 03 compliant.

> ### macOS
>
> Owing to its BSD root, many of the macOS commands behave in line
> with their BSD counterparts. Starting with Mac OS X 10.5 (_2007_)
> and up to at least 12.0 Monterey (_2021_) macOS has been certified
> as a UNIX 03 operating system; it is compliant with a set of
> standards called the Single Unix Specification (SUSv3) based on
> POSIX and other related standards.
>
> An environment variable `COMMAND_MODE` can be set to `legacy` to
> invoke Mac OS X 10.3 semantics for commands. When this variable is
> not set, SUSv3 complaint behavior can be expected.
>
> For the remainder of this post only POSIX and GNU `ps` will be
> considered.

The two most significant flags are to either select all users'
processes that are associated with a terminal or to select all
processes regardless of whether or not they have a terminal.

- `-a` is the POSIX standard option to select processes associated
  with terminals. Both System V and BSD have a `-a` and `a` option,
  respectively, but with slightly different semantics. System V omits
  session leaders, while BSD does not. This is specified as optional
  in the standard; GNU `ps` follows the System V model.

-  `-A` shows _all_ processes. It was chosen by POSIX as a compromise
   between the System V `-e` and the BSD `g` options. The POSIX XSI
   extension documents `-e` as equivalent to `-A`.

The BSD option `x` to show processes without a terminal can trace its
origin back to the very first version of `ps`. Any process that did
not have a "control typewriter" was listed with a terminal name of
`x`.

- `-d` shows all processes except session leaders. In practice this
  means users' interactive shells and parent dæmon processes will not
  show up.

Several options exist that take a comma- or space-separated list of
IDs to select processes.

- `-p proclist` selects all processes from the _proclist_.

Older versions of BSD `ps` allowed for specifying process IDs without
a flag. This is still supported in GNU and macOS `ps`.

- `-t termlist` selects all process associated with the terminals in
  _termlist_.

- `-U userlist` selects all processes with the real user given as an
  ID or name in _userlist_.
- `-u userlist` selects all processes with the effective user given as
  an ID or name in _userlist_. This is a POSIX XSI extension and
  whether the output is user IDs or usernames depends on whether the
  `-f` flag is used. This is in line with the System V behavior.

- `-G grouplist` selects all process with the real group of the
  creator of the process given as an ID or name in _grouplist_.
- `-g grouplist` logically should select process with the effective
  group ID, similar to how `-u` works. GNU `ps` will in fact work this
  way as long as at least one item in the _grouplist_ is
  non-numeric. The traditional System V definition is to select by
  process group list, POSIX defines it as selecting by session leader.

The unusual definition of the `-g` flag shows the complexity of `ps`
options. GNU `ps` solves this particular problem by supporting the
long options `--User`, `--user`, `--Group`, and `--group`.

Another difficulty is how different operating systems support process
groups and sessions. System V defines the `-g` option as being used
for process groups. The session concept originated in POSIX as a an
extension of System V and BSD 4.3 process groups and made it into
SVR4. POSIX defines `-g` as selecting by session leader instead of
process group, which is how GNU `ps` implements it. A useful output
control to list IDs for processes, process groups, and sessions is
`-j`, which is non-POSIX, but widely supported in practice.

Interestingly, macOS does support the session concept, but `ps` and
other tools cannot display the session ID.

# What to Show About the Processes

The POSIX standard suggests a default set of output fields: process
ID, terminal name, cumulative execution time, and command name. The
standard recognizes that defining different sets of fields is
difficult and introduced the `-o` option for user-defined format
control.

The only two predefined output options specified for POSIX `ps` are
the System V-derived `-f` and `-l`, which are the "full" and "long"
option, respectively. The default fields will be displayed with either
option.

The `-f` flag is more user-focused, similar to (but less extensive
than) the BSD `u` option. It also outputs the username and not the
UID. The `-l` flag includes more scheduling and memory management
fields. Because process start time and username are displayed with
`-f`, but not `-l`, it makes sense to list both with `-lf`.

The user-defined format control gives the most control over the output
of `ps`. The macOS and GNU `ps` implement a `-O` option, which _adds_
fields to the default list. POSIX only specifies `-o`.

The `-o format` option expects a single, possibly quoted _format_
string that specifies comma- or space-separated output fields and
allows for renaming (or elimination) of headers. Multiple `-o` options
can be used in one `ps` command.

The equivalent to the default set of fields is:
```
ps -o pid,tty,time,comm=CMD
```

Not all fields have the same name as their header. The field for the
command name, which has `CMD` as its header name, is `comm`. Using the
`=` allows for renaming; leaving the right-hand side empty leaves the
header blank.

The POSIX standard shows an example of spaces in header names, which
runs counter to the `-o` option allowing space-separated field
names. The example does not work for GNU or macOS `ps`.

# How to Format the Output

There are no formatting options specified in the POSIX standard for
`ps`. If the column width is insufficient ID fields will show numeric
instead of textual values.

While `ps` should determine terminal width automatically, it can be
overridden with the `COLUMNS` environment variable if need be. The
traditional BSD option for adjusting output width is `w`: one `w`
switches to 132 columns, more than one selects unlimited width.

To suppress the header, the `-o` option with empty renaming can be
used. GNU `ps` also has a `--no-headers` option. More portably, the
output of `ps` can be piped to `tail +2`, but this causes the screen
width to be ignored.

Some version of `ps` have sorting options. GNU `ps` has a `--sort`
option with the same field name syntax as `-o`.

GNU `ps` offers an ASCII art process tree with `--forest`. For a
visually more appealing `ps` output, the `pstree` command has many
options.

For continuously updating `ps` output, `top` or the more powerful
`htop` are the right tools.

# Conclusion

This post is only a small insight into `ps` options. Especially the
GNU version of `ps` supports many different options and by setting the
`PS_PERSONALITY` environment variable can emulate many different
systems' `ps` behavior. The manual pages `ps(1)` are the reference for
the command and the GNU version would amount to 20 pages printed.

It is very useful to familiarize oneself with the common `ps` options
for interactive use. Parsing the output programatically can be
challenging though and is rarely portable.
