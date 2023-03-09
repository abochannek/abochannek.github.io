---
layout: post
title:  "Emacs: Keyboard Macros (Part 2)"
date:   2023-03-09 00:40:29 -0800
categories: Emacs
related: [
	"Emacs: Auto-Save and Backup Files",
	"Emacs: Registers",
	"Emacs: Keyboard Macros (Part 1)"
	]

---

# Contents
- [Keyboard Macro Counters](#keyboard-macro-counters)
- [Querying During Macro Execution](#querying-during-macro-execution)
- [Macro Step Edit](#macro-step-edit)
- [Macros In Calc Mode](#macros-in-calc-mode)
- [Conclusion](#conclusion)

# Keyboard Macro Counters

Keyboard macros carry with them a counter that can be set,
incremented, reset, and inserted as part of the macro. A common use of
the counter is to create text that only varies by a numeric value. The
macro counter is automatically incremented by one (the default) after
each run of the macro.

A real-life example of the macro counter is in the answer to a
question that was posed on the `#emacs` IRC channel: How can keyboard
macros be used to build the followin `case` constructs for a `switch`
statement in C++?

```
case 1:
case 2:
case 3:
case 4:
case 5:
case 6:
```

This can be easily done with a keyboard macro. The most obvious way is
to start a macro with `C-x (` or `<F3>`, type the `case ` construct
followed by the counter insertion command `C-x C-k C-i` or `<F3>`
again, followed by the `:` and a newline and then end the macro with
`C-x )` or `<F4>`.

| `C-x C-k C-i` | Insert macro counter |

Each time the macro counter is inserted, it is incremented by default
by one. A prefix can be given to `C-x C-k C-i` to change the increment
value.

An issue in this example is that the default start value for a macro
counter `0` and i needs to be changed to `1`. The command for that is
`C-x C-k C-c` with the starting value as its prefix (or entered at the
prompt if there is no prefix.)

| `C-x C-k C-c` | Set macro counter |

The macro counter needs to be set _before_ executing the macro. If the
command is used during macro definition, the counter will be set to
the given value in each iteration.

As described in the [first part](../../01/31/emacs-kbdmacros.html) of
this post, macros can be executed directly at definition time. This
means that the macro definition and execution can be combined:

`C-u 1` `C-x C-k C-c` `<F3>` case `<SPC>` `<F3>` : `<RET>` `C-u 6` `<F4>`

The use of `<F3>` here has two different meanings, depending on
context. The first one starts the definition of the macro (equivalent
to `C-x (`) and the second one inserts the counter value (equivalent
to `C-x C-k C-i`.)

> #### Format Strings
>
> The default format for inserting counters is an integer number.
> However, a different format string can be assigned either globally or
> within a macro definition at definition time. This can make sense if
> either the numbers need to be padded or if there is a repeated text
> pattern in which the counter should be embedded.
>
> | `C-x C-k C-f` | Set format string |
>
> This presents an alternate approach to use a macro for the example
> above. If the counter should be right justified within three digits,
> the macro that uses the entire text as a format string is defined
> like this:
>
> `<F3>` `C-x C-k C-f` case `<SPC>` %3d: `<RET>` `<F3>` `<RET>` `<F4>`

During macro execution, the counter can be incremented or decremented
with the `C-x C-k C-a` command. The prefix determines the value by
which to change the counter.

| `C-x C-k C-a`       | Add to macro counter |

#### Resetting The Counter

During macro execution, the counter can be reset to two different
values:

| `C-u` `C-x C-k C-a` | Counter at the last insert                    |
| `C-u` `C-x C-k C-c` | Counter at the beginning of current iteration |

Additionally, using the prefix without a numeric value to the insert
command inhibits the increment of the counter.

| `C-u` `C-x C-k C-i` | Insert previous counter and don't increment |

Let's say the goal is to print a pattern like this:

```
112211
223322
```

This can be accomplished with this macro (using `<f3>` instead of `C-x
C-k C-i` for brevity.)

`C-u 0` `<f3>` `<f3>` `C-u 0` `<f3>` `<f3>` `C-u` `C-x C-k C-c`
`<f3>` `C-x C-k C-a` `<f3>` `RET`

This can be broken down into the following pieces:

| `C-u 0` `<f3>`      | inserts the counter without increment                |
| `<f3>`              | inserts the counter and increments                   |
| `C-u 0` `<f3>`      | inserts the counter without increment                |
| `<f3>`              | inserts the counter and increments                   |
| `C-u` `C-x C-k C-c` | resets the counter to the beginning of the iteration |
| `<f3>`              | inserts the counter and increments                   |
| `C-u` `C-x C-k C-a` | resets the counter to the last insert value          |
| `<f3>`              | inserts the counter and increments                   |
| `RET`               | next line                                            |

The pattern `<f3>` `C-u` `C-x C-k C-a` `<f3>` is used here for
demonstration purposes, it can be replaced with a `C-u` `<f3>` `<f3>`.

This example presents an interesting challenge. The `C-u` `C-x C-k
C-c` construct uses the macro counter from the most recent execution
of a macro. What that means is that if the macro counter is reset
between executions, the next execution still carries with it the
previous one's final counter value. The solution to this issue is to
either decrement the counter back to the original value or to consider
using [number registers](../../01/11/emacs-registers.html) instead.

# Querying During Macro Execution

A useful keyboard macro feature is the ability to stop and allow the
user to prompt whether to continue. The macro query command is used
for that.

| `C-x q` | Prompt user for confirmation |

The possible responses are:

| `y`/`<SPC>` | Continue macro                                    |
| `n`/`<DEL>` | Skip the rest of the macro and start the next one |
| `q`/`<RET>` | End macro execution                               |

Going back to the [example](#keyboard-macro-counters) at the
beginning of this post, a `C-x q` could be inserted after each line is
finished to query the user whether to continue or not:

`C-u 1` `C-x C-k C-c` `<F3>` case `<SPC>` `<F3>` : `<RET>` `C-x q`
`<F4>`

Running this macro with a large number prefix and then simply holding
the `<SPC>` key pressed would insert line after line until the user
decides to stop and quit the macro.

An alternate use of the query function is the ability to enter
recursive editing mode when pressing `C-r` at the prompt or prefixing
the command during macro definition as `C-u C-x q`.

A common use for this is to allow for some manual editing in the
middle of a macro execution because that change might be specific to
each execution, e.g., different text and not just a counter.

> #### Recursive Editing
>
> Recursive editing is a feature that dates back to the earliest days
> of TECO EMACS. The command to enter the real-time (i.e., full-screen
> display) editing mode that became EMACS from TECO was `C-r` and the
> interactive editing commands were referred to as _^R commands_. This
> real-time mode could be entered recursively; EMACS subsystems (like
> RMAIL for reading and editing mail) would call EMACS itself for
> editing functions.
>
> In modern Emacs, recursive editing is a convenient feature when
> doing a `query-replace` (`M-%`). It is also implicitly used by a
> number of modes like Calc or Edebug. Brackets (`[`…`]`) around the
> mode name on the Mode Line show that Emacs is currently in recursive
> edit mode. The number of brackets indicate the depth of levels of
> recursion.
>
> After editing is completed, recursive editing is exited by `C-M-c`
> or the command that called it is aborted with `C-]`. This recursive
> edit exit command is already mentioned in an _EMACS Introduction_
> manual from 1978.
>
> | `C-M-c`         | Exit recursive editing                           |
> | `C-]`           | Exit recursive editing and abort current command |
> | `M-x top-level` | Exit all recursive editing levels                |

# Macro Step Edit

[Macro
editing](../../01/31/emacs-kbdmacros.html#editing-and-naming-macros)
has been discussed in the previous post. Step editing not only offers
sophisticated debugging capabilities for keyboard macros, but can also
be used to edit and modify macros. The command to start step editing
for the most recent macro is `C-x C-k <SPC>`.

Step editing provides insight into the execution of a macro as the
user single-steps through it. The help text in the minibuffer shows
all the options that are available when requesting help with the `?`
key. The below is for the example at the beginning of this post.

```
Macro: case SPC <f3> : RET
--------------Step Edit Keyboard Macro  [?: help]---------------
 Step: y/SPC: execute next,  d/n/DEL: skip next,  f: skip but keep
       TAB: execute while same,  ?: toggle help
 Edit: i: insert,  r: replace,  a: append,  A: append at end,
       I/R: insert/replace with one sequence,
 End:  !/c: execute rest,  C-k: skip rest and save,  q/C-g: quit
----------------------------------------------------------------
Next command: kmacro-start-macro-or-insert-counter [yn iIaArR C-k kq!]
```

As always, the info manual is the definitive source of information.
Step editing is powerful and can help the novice user better
understand the construction and execution of keyboard macros.

# Macros In Calc Mode

Calc mode, the advanced Emacs calculator, uses keyboard macros to
implement programmability. Similar to how many scientific calculators
record sequences of instructions by recording keystrokes, Calc takes
advantage of the keyboard macro facility and extends it.

Defining a macro in Calc uses the standard key sequences. It
integrates the macro editing facility with its own user-defined key
naming approach to bind macros to a two-key sequence with the `z`
prefix.

Calc binds `C-x * m` to parsing a keyboard macro definition and
assigning it to the last macro. While it is a Calc keybinding, this
command is independent of Calc and a useful way to quickly create
macros from stored definitions.

Calc extends keyboard macros with its own commands to produce a
programming language. It implements these features:

  * Loops
  * Conditionals
  * Local Variables
  * Prompts

Standard keyboard macros do not have these capabilities.

# Conclusion

Keyboard macros have been in Emacs since well before GNU Emacs and are
a fundamental tool to improve the ease and accuracy of repeated
actions. At their most basic, they are a simple way to record and
recall sequences of keystrokes. Counters and queries add to the power
of macros and the different editing functions make them more
manageable. Libraries of macros can be built up with named macros.

It is not necessary to master all keyboard macro features to take
advantage of them. Even just the record and execute function is enough
to make a significant difference. Locating and extracting data from
multiple buffers, reformatting irregular text, or calling up a
sequence of commands for specific use cases are examples where
keyboard macros shine.
