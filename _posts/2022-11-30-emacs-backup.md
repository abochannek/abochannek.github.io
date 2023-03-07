---
layout: post
title:  "Emacs: Auto-Save and Backup Files"
date:   2022-11-30 02:10:15 -0800
last_modified: 2023-03-06 18:14:21 -0800
categories: Emacs
related: [
	"Emacs: Registers",
	"Emacs: Keyboard Macros (Part 1)"
	]

---

# Contents
- [Introduction](#introduction-to-the-emacs-series-of-posts)
- [Auto-Saving](#auto-saving)
- [Backup Files](#backup-files)
- [Conclusion](#conclusion)

# Introduction To The Emacs Series Of Posts

Emacs is more than just a powerful editor, it's a development
environment, file manager, email client, and much more. Because of the
hundreds of configuration options and endless ways to customize Emacs,
it can be quite confusing to set up Emacs.

The purpose of this series of posts is to focus on a basic,
fundamental feature of Emacs and explain it in practical terms. The
`info` manuals for Emacs and its modes are extensive and serve as the
authoritative documentation. These posts do not replace this
information, but instead provides a working knowledge of how to use
and configure a particular feature.

The behavior of a feature will be explained independent of how the
configuration variables are set. They can be set manually in the Emacs
init file (e.g., in `init.el`) or with `customize-mode`, which was
introduced with Emacs 20.1 (_1997_) to simplify setting options in
easy to find groups. Another approach are pre-configured Emacs starter
kits that are meant to make Emacs easier for new users (e.g., those
coming from Vim.)

# Auto-Saving

Auto-saving documents is intended to protect the user from power,
hardware, or software issues that prevent changes to documents from
written out to permanent storage. Even though it is widely implemented
in many applications, compulsively hitting a _Save…_ button (or in the
case of Emacs `C-x C-s`) whenever there is a brief pause is a common
user behavior.

In the 1980s, ads for microcomputer word processing software routinely
advertised it as an important feature to recover from system crashes
and power outages. There were even standalone auto-save tools that
operated independently of the applications.

The `ex` editor in 1BSD (_1977_) enhanced the standard Unix `ed`
editor with a recovery option. The manual explained its purpose:

> After a crash it is usually possible to recover the work you were
> doing to within a few lines of changes of where you were. This
> applies both to editor crashes and system crashes.

In the 2BSD manual gives another problem scenario:

> If the system (or editor) crashes, or you accidentally hang up the
> phone […]

Reliability of minicomputers in the 1970s was a challenge and software
problems not uncommon. An introductory book on Unix under the _Byte
Book_ imprint _Introducing The UNIX System_ goes to great lengths to
explain that `ed` can lose files during a crash and advises to save
frequently. It even goes as far as to suggest:

> This is particularly important when using the editor to create a new
> file, and is the reason for using `cat` (as we showed earlier) to
> create a new file by entering text directly, without an editor.

The book later goes explains file recovery in `ex`.

While hardware reliability has significantly increased in the decades
since then, cloud-based editors and word processors face a challenge
similar to the terminal phone line problem from the 2BSD `ex` manual.

Emacs had an auto-save feature since its earliest days at MIT (_1976_)
and likely inherited it from its predecessors that were built on TECO
as early as _1974_. GNU Emacs (_1985_) always had auto-save and its
functionality has not changed in principle in the decades since.

#### Auto-Save Files

By default, Emacs will auto-save visited files in the same location
where they were opened (with the exception of remote files.) The
auto-save filename is constructed by surrounding the original filename
with `#`. E.g., a `notes.org` file will be auto-saved as
`#notes.org#`.

Of course, Emacs allows to modify the filename pattern, to turn off
`auto-save-mode` altogether, and to force auto-saving non-file
buffers. The `info` manual documents that in detail.

#### Auto-Save Intervals

Two variables determine how frequently to auto-save:

  * `auto-save-interval` defaults to 300 characters (and no less than
    20)
  * `auto-save-timeout` defaults to 30 seconds of _idle_ time

In practice, auto-saving may not take place exactly at the given
timeout if the buffer is very large because of how long it can take to
save. Auto-save also happens as Emacs crashes or when an explicit
auto-save is performed (`M-x do-auto-save`).

When a buffer is saved (`C-x C-s`), Emacs removes the auto-save file
unless that feature is turned off. Killing a buffer (by default) does
not. Auto-save is also automatically turned off when "a substantial
part of the text in a large buffer" is deleted according to the
manual. What constitutes "substantial" is a hard-coded heuristic in
Emacs, which can be turned off, but not altered.

#### Auto-Saving The Visited File

Emacs offers a minor mode `auto-save-visited-mode`, which saves the
visited file _in addition_ to the auto-save file. The interval
settings for auto-save mode do not apply, instead the
`auto-save-visited-interval` variable can be set and defaults to five
seconds of idle time.

The `auto-save-visited-mode` replaced earlier functionality in Emacs
26.1 (_2018_) and in Emacs 29.1 (expected in _2023_) the
`auto-save-visited-predicate` can be set to determine which visited
files to auto-save.

> #### Lock Files Are Not Auto-Save Files
>
> A file looking similar to an auto-save file is the edit lock. It is
> usually a symbolic link of the form `.#notes.org` that points to a
> synthetic file with the username, hostname, and Emacs process ID as
> its name.

#### Recovery

Individual files can be recovered with the `M-x recover-file` command,
which prompts for a filename. If the auto-save file is newer than the
saved file, directory entries for both are shown and the user is asked
if they want to recover the file. The auto-save file is then read in
and can be saved under the original filename.

Emacs can also recover all auto-save files after a crash with `M-x
recover-session`. By default, a list of all visited files and their
auto-save files are recorded in a file in the
`~/.emacs.d/auto-save-list` directory. The filename for the auto-save
list includes the hostname and Emacs process ID. After the file is
chosen, Emacs will prompt for all existing auto-save files listed in
the same manner as the single file recovery.

While not strictly related to auto-saving files, `desktop-save-mode`
makes restoring a previous session easier. Emacs will save information
about all open files, window and frame configuration, and buffer
positions during an orderly shutdown if it is enabled. By default,
this information is read on start-up from the
`~/.emacs.d/.emacs.desktop` file.

# Backup Files

Backing up files is an _insurance_ against different failure
scenarios, some of which overlap with auto-saving. When a backup file
is on the same system and storage as the original, it cannot really
prevent data loss in case of catastrophic hardware failures, software
bugs, or malicious acts. Off-line storage (or these days cloud-based
backups) can help here. The Emacs file backup strategy primarily helps
recover from accidental changes and deletions by the user.

When saving a modified file, Emacs will rename the original file to
its backup name (by default appending a `~`, so `notes.org` turns into
`notes.org~`) and save the new content under the original file name.
Whether Emacs should rename or copy the file follows a somewhat
complicated, but generally reliably set of rules and can be controlled
by several variables – the `info` manual is (as always) the
authoritative source for it.

As long as the file remains visited, no new backup will be saved even
if the backup file has been deleted. Only upon opening the file again
either after the buffer has been killed or Emacs has been restarted,
will a new backup file be written after the first save.

Saving a file with `C-u C-x C-s` is equivalent to saving the file the
first time after it has been reopened, i.e., a backup will be made the
_next_ time the file is saved. Saving with `C-u C-u C-x C-s` creates a
backup file and saves the current buffer at the same time, which is
likely what a user would expect. A triple-`C-u` combines both of the
above functions.

Forcing backups can be a useful when significant or repeated changes
are made to a file, but the buffer has not been killed. It is worth
considering whether in this case [version control](#version-control)
may not be a better alternative or a supplemental approach to using
backup files. An easy way to achieve some basic version control
without setting up a version control system is to use [Numbered
Backups](#numbered-backups). In this scenario, forcing writing a
backup file is similar to committing a file in a version control
system.

Backup file creation can be disabled altogether, disabled for certain
directories (temporary directories by default), or they can be created
in specific backup directories.

#### Version Control

When Emacs recognizes that a file is tracked in a version control
system, it will not write a backup file when saving it. The version
control systems Emacs can identify are SCCS/CSSC, RCS, SRC, CVS,
Subversion, Git, Mercurial, Bazaar, and Monotone. If the
`vc-make-backup-files` variable is set to something other than `nil`,
backup files will still be created, which may be a sensible options
depending on how frequently changes are checked into version control.

#### Numbered Backups

In the absence of a version control system, many users who want to
keep multiple edited versions of a file around, will manually change
the filename to include a sequence number. Several operating systems
used to support automatic versioning of files at the file system
level; DEC's VMS (introduced in _1977_) is one of those systems and is
still in use. Its ODS-2 file system supports version numbers natively
and an example of a filename is `MYFILE.TXT;4`, the fourth saved
version of the file `MYFILE.TXT`.

An earlier operating system, also from DEC, and which has a close
connection with the early history of Emacs is TOPS-20. It too
supported file versions (called _generations_ there) in the form of
`MYFILE.TXT.4`. TOPS-20 in turn inherited this feature from BBN's
TENEX, on which it was based. An early proposal for TENEX (then called
Ten-Sys) from _1969_ already discussed optional file versions. It's a
useful feature that unfortunately has not survived in more recent
operating systems.

GNU Emacs got numbered backups very early in its life with Emacs 17
(_1985_).

> #### Snapshots
>
> A number of file systems offer built-in snapshot capabilities that
> usually allow user-initiated recovery of files based on a point in
> time. Examples are Btrfs on Linux, ZFS on a number of Unix and
> Unix-like operating systems, and Apple's APFS for macOS.

#### Numbered Backup Files

The creation of numbered backup files in Emacs follows the same
approach as single backup files, except that the naming pattern is
different: `notes.org~` becomes `notes.org.~1~`, `notes.org.~2~`, and
so on.

The somewhat confusingly named variable `version-control` controls
numbered backups in the following way:

| `nil` (default) | Make single backups unless there are already numbered ones |
| `t`             | Numbered backups                                           |
| `never`         | Single backups                                             |

While the variable is global, it can be set locally and it may make
sense to set it as a directory local variable.

#### Deleting Backups

Numbered backup files can be automatically deleted; numbered and
single backup files, as well as auto-save files can easily be manually
deleted in `dired` mode.

Three variables determine how many numbered backup files to keep:
`kept-old-versions`, `kept-new-versions`, and `dired-kept-versions`
(all default to 2.) While `kept-new-versions` is self-explanatory,
`kept-old-versions` behaves somewhat surprisingly. With their default
settings of 2, a sequence of five independent saves for a file
`notes.org` would result in these files, the current and four numbered
backups:

```
notes.org.~1~
notes.org.~2~
notes.org.~3~
notes.org.~4~
notes.org
```

The next time, `notes.org` is saved, `notes.org.~3~` is deleted
because the oldest and the newest two backups, respectively are kept.
Emacs will prompt for the deletion of excess backups when the
`delete-old-versions` is set to `nil` (default.)

```
notes.org.~1~
notes.org.~2~
              ← notes.org.~3~ has been deleted
notes.org.~4~
notes.org.~5~
notes.org
```

The `kept-old-versions` therefore determines how many of the
_earliest_ backups to keep, which may not be very useful for a file
that has been edited repeatedly over long periods of time.

In `dired` mode, the shortcuts for flag for deletion are:

| `#` | auto-save files      |
| `~` | any backup files     |
| `.` | excess numeric files |

The subtle difference is that the value of `dired-kept-versions` is
used instead of `kept-new-versions`. To execute the deletion of the
flagged files, use `x` in `dired` mode.

#### Example

The defaults Emacs offers are sensible. The following example
configuration offers an alternate approach.

```
(setq
 version-control t
 delete-old-versions t
 kept-new-versions 10
 kept-old-versions 0)
```

These settings result in numbered backups with the ten most recent
saves of a file to be kept and none of the original ones. It also
causes Emacs to delete excess numbered backups to be deleted silently.

Using the `.` command in `dired` mode deletes all but the last two
numbered backups, which is the default.

# Conclusion

Setting up Emacs is an iterative process. There is no one right setup
and it's easy to change settings to fit changing needs. Understanding
the underlying functionality is critical to take full advantage of
what Emacs has to offer.
