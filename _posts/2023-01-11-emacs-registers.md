---
layout: post
title:  "Emacs: Registers"
date:   2023-01-10 21:17:52 -0800
last_modified: 2023-01-30 23:07:39 -0800
categories: Emacs
related: [
	"Emacs: Auto-Save and Backup Files",
	"Emacs: Keyboard Macros (Part 1)"
	]

---

# Contents
- [What Are Registers?](#what-are-registers)
- [Emacs Register Origins](#emacs-register-origins)
- [Using Emacs Registers](#using-emacs-registers)
- [Clearing Registers](#clearing-registers)
- [Bookmarks](#bookmarks)
- [Summary](#summary)

# What Are Registers?

Registers are typically associated with fast-access memory locations
inside a computer's processor. They are a fundamental part of a
computer's instruction set architecture. More generally, they are a
storage location for data and can be found even in mechanical and
electro-mechanical calculation machinery that predate programmable
electronic digital computers.

The memory function of pocket calculators can be thought of as
manipulating a register and programmable desk calculators of the 1970s
often had dozens of memory locations referred to as registers. One of
the oldest Unix programs is the `dc` desk calculator (_1970_). Both
numbers and strings can be stored in registers with the `s` command
and their content loaded again with the `l` command. Registers are
named by a single letter, which is similar to how programmable
calculators and indeed Emacs work.

# Emacs Register Origins

EMACS, in its original form as a TECO extension, got register
capabilities by virtue of TECO's _q-registers_. TECO was created by
Dan Murphy while a student at MIT in 1962 on the PDP-1[^1]. Because of
resource constraints, interactive editing was not really feasible at
the time, which is why TECO was designed to work off of a command
tape, more like the Unix `sed` stream editor a decade later.
Programmability was added to TECO quickly and first text variables and
then loops were implemented. Text variables were designated with a
single character as were integer variables that were used for looping.
The `q` command retrieved the _quantity_ of a register, which is why
they were referred to as _q-registers_.

The TECO manual describes the _q-registers_ (PDP-11 TECO, _1980_):

> TECO provides 36 data storage registers, called Q-registers, which
> may be used to store single integers and/or ASCII character strings.
> The Q-registers have one character names: A through Z and 0
> through 9. Each Q-register is divided into two storage areas. In the
> numeric storage area, each Q-register can store one signed integer.
> In the text storage area, each Q-register can store an ASCII
> character string which may be either text or a TECO command string.

EMACS, the TECO-based version that ran on ITS and TWENEX, made heavy
use of _q-registers_. The `C-X X` and `C-X G` commands to put text
into and get text out of a register, respectively, are directly
analogous to TECO's `x` and `g` command. Those commands were carried
over as key bindings into GNU Emacs until the prefix changes in Emacs
19 (public release in _1994_).

Interestingly, Gosling Emacs, the direct precursor to GNU Emacs, did
not have a register feature while GNU Emacs did since the beginning.

# Using Emacs Registers

#### Register Command Prefix

The work on GNU Emacs began in 1984 and before the public release of
Emacs 13 (_1985_), both rectangle and register commands were
implemented. The register commands were carried over from the earlier
EMACS and used the same command structure as described above. The
rectangle commands used the `C-x r` prefix.

In Emacs 19, `C-x` key bindings were cleaned up a bit. Besides
narrowing and abbrev commands, the register and rectangle commands
were unified under `C-x r`. Those key bindings have not been
fundamentally changed in the three decades since.

#### Register Names And Types

Registers are named with a single character. It does not need to be an
ASCII alphanumeric character, it can be any character valid in Emacs,
except the ones used for quitting (i.e., `C-g`.) The placeholder for a
register name in the command descriptions is _`R`_.

There are two general families of registers, location and content
registers (these terms are used in this post for the purpose of
explanation, they are not used in the `info` manual, which is as
always the authoritative source.)

Locations are used to record and jump to points in buffers as well as
to restore files and display configurations. Content registers are an
alternative to the kill ring in that their content does not change
after another kill.

#### Register Command Structure

The key binding to save data in a register is `C-x r` followed by one
letter for the type and then the register name _`R`_.

#### Location Registers

| Letter  | Type     | Purpose                                   |
|:--------|:---------|:------------------------------------------|
| `<SPC>` | Position | Point in current buffer                   |
| `w`     | Window   | Window configuration in the current frame |
| `f`     | Frame    | All frames and their windows              |
| n/a     | Buffer   | Buffer name                               |
| n/a     | File     | File name                                 |

In practice, this means to save the position of the point in the
current buffer, for example, the command is `C-x r` followed by the
_Space_ key and then a single letter for the register name.

It's good practice to use a mnemonic for the register name, but
considering it can only be a single letter that may prove difficult.
Let's say the window configuration for several IRC channels related to
Emacs (e.g., `#emacs` and `#gnus`) on _Libera.Chat_ should be saved in
order to easily jump back to that view. A good mnemonic for this might
be the register name `E`, so the key sequence to save that window
configuration then is `C-x r w E`.

Underneath the key binding, the Elisp function that is used to save
registers is called `set-register`. It takes a register name and the
information that is supposed to be saved. In the case of a file or a
buffer name, there is no default key binding and that function needs
to be called explicitly. The example from the `info` manual for saving
a file name in a register (in this case the register name `z`) is:

`(set-register ?z '(file . "/gd/gnu/emacs/19.0/src/ChangeLog"))`

[Bookmarks](#bookmarks) are a better way to record files and their
point location.

These "location" registers are identical in how their content can be
recalled. In order to _jump_ to a location in a register, the easy to
remember key binding is `C-x r j` followed by the register name.

A special key binding to store keyboard macros is available as `C-x
C-k x` followed by the register name; executing the macro in a
register is accomplished by "jumping to it" with `C-x r j`.

> #### Register Preview
>
> When jumping to a register (or inserting its content, see below), a
> window with the current register assignment will pop up after a
> default time of one second before entering the register name. That
> time can be customized and this preview can also be requested
> without jumping to a register. The `info` manual has all the
> additional information.
>
> It's important to note that the register preview will show _all_
> registers, regardless of what they contain. That means that if a
> register contains text, but the preview appears after a `C-x r j`,
> that content cannot be inserted and an error will be displayed. The
> same is true in the opposite scenario when a location register is
> selected after a `C-x r i`.

#### Content Registers

| Letter | Type          | Purpose           |
|:-------|:--------------|:------------------|
| `s`    | Text (String) | Current region    |
| `r`    | Rectangle     | Current rectangle |
| `n`    | Number        | Number            |

Storing text and numbers is why registers in TECO and then Emacs were
implemented. Text (strings) between the mark and the point are saved
with `C-x r s` and when used with the universal `C-u` prefix the
selected text is also deleted.

The rectangle between the mark and the point is saved into a register
with the `C-x r r` command followed by a register name. The command to
_insert_ a register's content into the current buffer at point is `C-x
r i` followed by the register name.

> #### `C-x r` And Rectangles
>
> Because register and rectangle commands were part of the prefix
> clean-up in Emacs 19, there are many additional key bindings that
> start with `C-x r`. Most of the key bindings are fairly mnemonic and
> the overlap doesn't really contribute to confusion. The `info`
> manual is the authoritative source.
>
> One obvious conflict is `C-x r s` to store text in a register and
> `C-x r t` to fill a rectangle with a string. Once that inconsistency
> is apparent though, remembering the difference is not difficult.

The number to be stored in a register is assigned with the universal
prefix `C-u`. E.g., to store the number `42` in register `H`, the key
combination is `C-u 42 C-x r n H`. Without the prefix, the number `0`
is stored in the register.

The increment command `C-x r +` applies to both text as well as number
registers. In the case of number registers, the content will be
increased by `1` unless there is a prefix.

For text registers, the increment is a an _append_ function. A prepend
function also exists and the append can be customized. The append
function can be used to build up text segments from different buffers
for later, potentially repeated inserts.

# Clearing Registers

Somewhat surprisingly, there is no default key binding to clear
individual registers. They can be set to `nil` with the `set-register`
function.

An easier way to manage register content is the `register-list`
package available on ELPA, which offers register editing, deletion,
and concatenation commands in an overview buffer similar to what is
available by defaults for Bookmarks.

# Bookmarks

Registers are saved in the default file `‘~/.emacs.d/.emacs.desktop`
and restored for the next Emacs session when `desktop-save-mode` is
enabled. Bookmarks are a better way to store and manage file locations
longer-term.

The main bookmarks commands are:

- `C-x r m` Sets the bookmark for the current file
- `C-x r b` Jump to bookmark
- `C-x r l` List all bookmarks

Bookmark names default to the filename, but can be given any other
name. At exit time of Emacs, bookmarks are by default saved in
`‘~/.emacs.d/bookmarks`. Bookmarks can be annotated, automatically
saved, and customized in other ways.

The bookmarks list command (`C-x r l`) shows all bookmarks and offers
comprehensive functions similar to `dired`. The mode description (`?`)
shows the mode commands, which are also available on the menu bar.

# Summary

Registers are a great way to temporarily store numbers and text, point
locations, and window and frame configurations. Text registers can
function as an alternative to the kill ring to build up text snippets,
number registers can be used as counters.

The key bindings are all prefixed with `C-x r`, which is also used for
rectangles. The command names are easy to remember and well organized.

Together with the Desktop feature, registers are saved across
sessions. Bookmarks are a better way to save filenames and point
locations and come with a powerful editing mode.

[^1]: D. Murphy, "[The Beginnings of
    TECO](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=5370787),"
    in IEEE Annals of the History of Computing, vol. 31, no. 4, pp.
    110-115, Oct.-Dec. 2009, doi: 10.1109/MAHC.2009.127.
