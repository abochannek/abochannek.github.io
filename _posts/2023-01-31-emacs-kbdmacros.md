---
layout: post
title:  "Emacs: Keyboard Macros (Part 1)"
date:   2023-01-31 01:52:33 -0800
last_modified: 2023-02-01 22:21:08 -0800
categories: Emacs
related: [
	"Emacs: Auto-Save and Backup Files",
	"Emacs: Registers"
	]

---

# Contents
- [Emacs Keyboard Macros Origins](#emacs-keyboard-macros-origins)
- [Defining And Executing Macros](#defining-and-executing-macros)
- [Editing And Naming Macros](#editing-and-naming-macros)

# Emacs Keyboard Macros Origins

The purpose of keyboard macros is to make repeated entry of commands
easier and less failure-prone. They are different from text expanders,
which Emacs offers with the `abbrev` functionality.

Macro recorders are a common feature of different applications as well
as operating systems. They developed alongside
"programming-by-example" research in the mid-1970s.

Richard Stallman (rms) encountered the full-screen editor _E_ while
visiting Stanford's AI Lab in 1976 and its design influenced EMACS.
Recording keyboard macros already existed in _E_:

> 10/7/75 -- Macro defining and calling commands: ⊗XDEFINE ⊗Y.  
> E now has a very simple yet useful macro facility which saves the
> user from having to type the same sequence of commands several
> times.

The _EMACS Manual for ITS Users_ ([AI Memo
555](https://dspace.mit.edu/handle/1721.1/6329), _1981_) shows how to
create keyboard macros in the "EMACS command language" with the
`KBDMAC` library. TECO EMACS did not use Lisp as an extension language
and keyboard macros were easier to compose than writing TECO macros:

> Keyboard macros differ from ordinary EMACS commands, in that they
> are written in the EMACS command language rather than in TECO. This
> makes it easier for the novice to write them, and makes them more
> convenient as temporary hacks. However, the EMACS command language
> is not powerful enough as a programming language to be useful for
> writing anything intelligent or general. For such things, TECO must
> be used.
>
> EMACS functions were formerly known as macros (which is part of the
> explanation of the name EMACS), because they were macros within the
> context of TECO as an editor. We decided to change the terminology
> because, when thinking of EMACS, we consider TECO a programming
> language rather than an editor. The only "macros" in EMACS now are
> keyboard macros.

Keyboard macros in GNU Emacs therefore originated in the earlier TECO
EMACS and the very same key bindings for defining and executing them
continue to be available. The distinction between keyboard macros and
extensions written in the underlying programming language (Elisp in
the case of GNU Emacs) also continues to be true.

# Defining And Executing Macros

The easiest way to define and use keyboard macros is what the info
manual refers to as "old style." In Emacs 22 (_2007_) the keyboard
macro capabilities were significantly enhanced by the then new
`kmacro` package. This will be discussed below.

The main keybindings (which go back to the original TECO EMACS) are:

| Key Sequence | Function                      |
|:-------------|:------------------------------|
| `C-x (`      | Start a macro                 |
| `C-x )`      | End a macro                   |
| `C-x e`      | Execute the most recent macro |

It is easy to use these commands for quick, one-off editing tasks. As
an example, let's assume a buffer with this content (in this example,
the underlined letter shows where the cursor, the _point_ is):

```
f̱oo bar baz quux
foo bar baz quux
foo bar baz quux
foo bar baz quux
```

If the goal is to uppercase the first word on the first line, the
second on the second line, etc. this can be done easily with a macro.

`M-u` uppercases a word, `C-n` (or `↓`) moves to the next line:

```
FOO bar baz quux
foo ̱bar baz quux
foo bar baz quux
foo bar baz quux
```

These two commands can be combined into a macro:

`C-x (` `M-u` `C-n` `C-x )`

and then executed with `C-x e`. After three additional executions, the
end result will look like this:

```
FOO bar baz quux
foo BAR baz quux
foo bar BAZ quux
foo bar baz QUUX
̱
```

Notice that the cursor, the point, has moved to the next empty line
because of the `C-n` in the macro.

#### Repeated Macro Execution

Executing a macro repeatedly can be accomplished in two ways, either
by hitting `e` after the `C-x e` to execute the macro one more time,
or by prefixing `C-x e` with a number to indicate how many times to
execute it. The `e` repeat only works immediately after a macro
execution and will terminate if it encounters an error.

As is true in general for Emacs, the universal prefix `C-u` stands for
the number four, multiple `C-u` multiply by four, i.e., `C-u C-u C-x
e` executes the most recent macro sixteen times.

A shortcut to executing a macro is ending the definition with `C-x e`
instead of `C-x )` which immediately executes it. Because it also
takes a prefix, the above macro could have been defined and executed
on all following three lines with this sequence:

`C-x (` `M-u` `C-n` `C-u 3` `C-x e`

Repeat macro execution at the time of definition works subtly
differently when using `<F4>`. `C-x e` end the definition of the macro
first and then executes the macro as many times as the prefix given.
`<F4>` on the other hand runs the macro _total_ number of prefix
times. In other words, to run the macro four times total at the end of
the definition, the numeric prefix for `C-x e` needs to be `3` and for
`<F4>` it needs to be `4`.

Macro execution on a _region_ has some special semantics. The command
`C-x C-k r` runs a macro on a line and then moves the point to the
beginning of the next line.

> # "New" Commands
>
> Since Emacs 22 (_2007_), a new `kmacro` package defined different
> key bindings and additional features for keyboard macros.
>
> | Old Keys | New Keys |
> |:---------|:---------|
> | `C-x (`  | `<F3>`   |
> | `C-x )`  | `<F4>`   |
> | `C-x e`  | `<F4>`   |
>
> Because function keys may behave differently on the platforms that
> Emacs supports, it is important to determine if the function keys
> are available to Emacs before using them for keyboard macros.

# Editing And Naming Macros

Emacs keyboard macros use the keyboard commands and not the underlying
functions bound to those keys. Editing macros therefore is editing
those keyboard commands. The `C-x C-k C-e` key sequence brings up a
buffer with the definition of the most recent macro, which can then be
edited right there. If for example an additional cursor movement
forward is required, simply putting a `C-f` on a new line of the macro
adds it to the definition. `C-c C-c` ends the editing of the macro.

Appending to the most recent macro can be done by prefixing either
`C-x (` or `<F3>`. A single `C-u` universal prefix first _executes_
the macro before then appending to it. A dual `C-u C-u` will not
execute the macro first. This behavior depends on the variable
`kmacro-execute-before-append` which defaults to `t` and can be
changed.

Macros can be named and stored as well as bound to a key sequence or a
register.

| Key Sequence | Function                   |
|:-------------|:---------------------------|
| `C-x C-k b`  | Bind macro to key sequence |
| `C-x C-k n`  | Assign name to macro       |
| `C-x C-k x`  | Store macro in register    |

Some key sequences are reserved for macros to avoid collision with
other key bindings: `C-x C-k` followed by the numbers `0` through `9`
and the letters `A` through `Z`, similar to how
[registers](../../01/11/emacs-registers.html) are named. If any of
those keybindings are used, only the single digit or letter needs to
be given when binding the macro.

The below tables shows the difference between storing the most recent
macro in a key sequence and a register, both using `U` in this
example.

|              | Store Macro   | Call Macro  |
|--------------|:--------------|:------------|
| Key sequence | `C-x C-k b U` | `C-x C-k U` |
| Register     | `C-x C-k x U` | `C-x r j U` |

In order to save macros across sessions named macros are used. When a
name is assigned to a keyboard macro, it becomes and interactive Elisp
function. After naming the macro `upper` with `C-x C-k n upper`, it
can then be called with `M-x upper`.

To save keyboard macros in the `init.el` file, simply open it up and
use `M-x insert-kbd-macro` to insert the macro as Elisp code into the
buffer to save it across sessions.

# Coming Up...

This post covered the basics of keyboard macros. Some more advanced
features are addressed in the second part: the keyboard macro ring,
counters and queries, recursive editing, and the interactive step-wise
editor. They offer additional capabilities that are worth exploring
even if they may be used less frequently. Counters and queries in
particular introduce some added flexibility to macro execution.
