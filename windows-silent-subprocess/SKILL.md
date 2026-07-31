---
name: windows-silent-subprocess
description: "Stop spawned processes from popping console windows on Windows. A console child steals keyboard focus mid-keystroke and can cause the user to lose work. Use whenever an agent, script, scheduled task or background service spawns a subprocess on Windows, and whenever a user reports windows flashing or focus being stolen."
---

# Spawning processes on Windows without stealing the user's focus

On Windows, spawning a console program from a script can open a **visible console window** that takes
keyboard focus. If the user is typing when it appears, their keystrokes go to the new window instead
of wherever they were working, and a stray Enter into a surprise console is its own hazard. All of
it caused by a background task they did not ask about and cannot see.

This costs nothing to prevent and is almost always overlooked, because the ecosystem's defaults were
written on macOS and Linux where the problem does not exist.

## The mental model, which is the opposite of what people assume

A console process **attaches to its parent's console if one exists**. If the parent has **no** console
: a windowless host, a service, a scheduled task, a GUI app, the child cannot attach, so Windows
**allocates a brand-new visible console for it**.

> Making the parent windowless is what *causes* the popups.

This is why "I made it run in the background and now windows flash" is such a common report. The fix
is not to hide the parent; it is to tell each child not to create a console at all.

## The fix

Pass the creation flag that suppresses console allocation. In Python:

```python
import subprocess, sys

def run(cmd, **kw):
    if sys.platform == "win32":
        kw["creationflags"] = kw.get("creationflags", 0) | subprocess.CREATE_NO_WINDOW
    return subprocess.run(cmd, **kw)
```

Two things make this work in practice:

- **`CREATE_NO_WINDOW` gives the child a hidden console, and grandchildren inherit it.** So one flag
  at the top of a process tree keeps the whole tree silent: you do not need to find every nested
  spawn. (A grandchild that explicitly asks for a new console still gets one; inheritance is a
  default, not a cage.)
- **OR the flag in; do not assign it.** Overwriting `creationflags` silently discards a caller's own
  flags, which is a subtle bug in a helper everyone routes through.

**Route every spawn through one helper** rather than remembering the flag at each call site.
Remembering does not scale, and the failure is invisible on the machine of whoever writes the code if
they are not on Windows. If you can, add a check that fails the build on a raw `subprocess.run(` or
`Popen(` outside that helper: the rule then holds without anyone maintaining it.

## The other sources of surprise windows

- **Scheduled tasks:** the "Hidden" checkbox hides the task from the UI list; it does **not** hide the
  window. Either run the task as "whether the user is logged on or not" (non-interactive, so nothing
  can display), or point the action at a windowless interpreter binary directly.
- **`.bat` / `.cmd` wrappers** flash a console before doing anything. Call the executable directly.
- **Shell helpers that launch programs** (for example PowerShell's `Start-Process`) open a new window
  by default; use the no-new-window option, or invoke the executable so it inherits the hidden
  console.
- **Opening a file by its type association** launches whatever application is registered, in a window
  you do not control. Drive the tool directly instead.
- **Plotting and imaging libraries** may open a viewer window. Select a non-interactive backend and
  write to a file.

## Two traps that follow

**A windowless parent has nowhere to write.** Two different failures, and they look nothing alike:

- Under a GUI-subsystem interpreter, `sys.stdout` may be `None`. In Python 3, `print()` then does
  **nothing at all**: no error, no output. Your logs simply vanish, which is worse than a crash
  because nothing reports it.
- A console-subsystem interpreter running detached can have a stdout backed by an invalid handle. There
  `print()` **raises**, and usually somewhere unrelated to the actual problem.

Log to a file, and do it from the start. Both failure modes argue for it, and neither announces itself.

**An interactive prompt in a hidden console hangs forever, invisibly.** Nothing can answer it and
nobody can see it. Always pass the non-interactive flags a tool offers, and always set a timeout.

## Why it deserves a rule rather than a fix

The failure is asymmetric: the person who writes the spawn usually never sees the window, because
their process has a console and the child quietly attaches to it. The person who suffers it is the
user running the same code from a service, a scheduled task, or an agent. That gap is exactly the
shape of a defect that survives review, so make it structural: one helper, enforced, rather than a
thing everyone is expected to remember.
