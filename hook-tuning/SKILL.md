---
name: hook-tuning
description: "Decide whether an automated guard should BLOCK or WARN. Block only on deterministic facts; warn and log on judgment. Use whenever you write a hook, pre-commit check, CI gate, lint rule or policy check; whenever a guard fires on something legitimate; and whenever you are tempted to add a bypass flag or to use one."
---

# Tuning a guard so it stays respected

**A false positive in a blocker is not an annoyance. It is a defect in the guard**, because it teaches
the people it protects to bypass it, and once bypassing is a reflex, the guard has stopped working
while still reporting that it is on.

That is the whole idea. The rest is how to act on it.

## Block or warn: the deciding question

> **Can this check be wrong about a legitimate case?**

- **No, it is a deterministic fact** → BLOCK. A missing required field. A syntax error. A file that
  does not parse. A checksum mismatch. An exact-format match for a credential: a known token prefix,
  or any file on a declared secrets path. These have no defensible counter-example, so blocking costs
  nobody anything. Note the care in that last one: *detecting* a secret by shape is a heuristic and
  belongs in the warn column; matching a format that only ever means one thing does not.
- **Yes, it involves judgment** → WARN, and LOG. Style. Naming. "This looks like it might be a
  secret." "This commit seems large." "This wording may be unclear." Every one of these has a
  legitimate exception, and the exception will arrive on a day when someone is in a hurry.

The test is not "how bad is the thing I am catching". It is **"how confident can this code be, in the
worst legitimate case"**. A serious problem detected by a heuristic still belongs in a warning,
because a heuristic that blocks will eventually block something correct, and that is the moment the
bypass habit is born.

## Why bypass flags corrode

Once a blocker has a `--force`, three things happen in order:

1. Someone uses it for a genuine false positive. Correct, and fine.
2. It becomes the known workaround for that check.
3. It becomes the reflex for *every* firing of that check, including the true ones.

By stage 3 the guard is decorative. Nobody decided that; it emerged from the friction.

So if you must ship an override:

- **Require a written reason**, not a bare flag. Reasons are read; flags are typed.
- **Log every use**, with the reason, somewhere that gets looked at.
- **Watch the log, and treat growth as a signal about the GUARD.** A rising override count means the
  check is miscalibrated, not that people are careless. Fix the check.
- **Never let a bypass be the path of least resistance.** If it is faster to override than to comply,
  the outcome is decided.

A guard with no override at all is legitimate when the check is genuinely deterministic, and that is
another reason to keep blockers deterministic: only they can be made unbypassable without cost.

## Warnings need to be worth reading

A warning nobody reads is as useless as a blocker everyone bypasses, and it fails more quietly.

- **Say what to do, not what happened.** "Line 40 uses X" is a fact. "Line 40 uses X; use Y instead
  because Z" is actionable.
- **Do not warn about the same thing forever.** A permanent warning is background noise within a
  week. Either fix it, ratchet it, or delete the rule.
- **Ratchet rather than nag.** Record today's count as a baseline and fail only when it *rises*. That
  turns an unfixable backlog into a rule that prevents new instances immediately, which is most of
  the value at none of the cost.

## Make the message do the work

When a guard fires, the person reading it is usually mid-task and does not know why this rule exists.
The message is your only chance:

- **name the specific thing** that tripped it, with a location;
- **state the fix**, concretely enough to apply without a search;
- **say why the rule exists**, in one clause: a rule with a visible reason gets respected; a rule
  that seems arbitrary gets routed around;
- **say how to proceed legitimately**, if there is a legitimate way.

```
REFUSED: config.yaml has no `timeout` (line 12).
  Add `timeout: <seconds>`. Without it the client waits forever, so the
  job hangs instead of failing -- and a hang is far harder to notice.
```

## When a guard fires on something legitimate

Treat it as a bug report about the guard.

1. **Do not just override.** The override is the symptom.
2. **Decide whether the rule is wrong or the boundary is wrong.** Usually the boundary: the rule is
   right but its scope catches a case nobody considered.
3. **Fix the check, then re-run it** on the case that tripped it and confirm it now passes, and on a
   genuine violation and confirm it still fails. Both directions, or you have not tuned it: you have
   turned it off.
4. **If a rule needs frequent exceptions, it is the wrong rule.** Narrow it until it is right, or
   demote it to a warning.

## The failure this skill is really about

Guards fail in two directions, and only one of them is visible.

A guard that is **too loud** is annoying, gets complained about, and gets fixed. A guard that has been
**routinely bypassed** looks healthy from the outside: it is installed, it runs, it reports. Nobody
files a bug against it. It just no longer changes any outcome.

Calibration is what keeps a guard in the first category. That is why a false positive in a blocker
deserves the same urgency as a missed detection: both end with the guard not working, but only one of
them tells you.
