# Maintainer's 05_ACCESSIBILITY_NOTES.md
## Overview

This document is written for anyone who contributes to, forks, or
collaborates on this repository. It describes accessibility
considerations that shaped how this documentation is written and how
this project is maintained. The general guidelines outlined were used to
help Brave Leo AI help Mir during the entire build process.

Reading this document will help you communicate and collaborate more
effectively with the primary maintainer.

---

## About the Maintainer

This project is maintained by Mir (Miranda). Along with periodic
paralysis, Mir has two documented physical considerations that affect
how she works with technical documentation and command-line
environments. None are constant — all are unpredictable in timing and
severity. All are real and should be factored into any collaborative work.

---

## Visual: Asteroid Hyalosis

**What it is:** Asteroid hyalosis is a condition where calcium-lipid
deposits accumulate in the vitreous humor of the eye, causing
persistent visual interference — similar to looking through a snow
globe that never fully settles. Mir has this in both eyes, with the
left eye more severely affected.

**What this means in practice:**

Certain character pairs are visually ambiguous and easy to confuse
at a glance:

| Pair | Example confusion |
|---|---|
| `rn` vs `m` | `rn` in a small font reads as `m` |
| `1` vs `l` vs `I` | Numeric one, lowercase L, uppercase i |
| `0` vs `O` | Zero vs uppercase letter O |
| `,` vs `.` | Comma vs period in dense output |

**Guidelines for contributors:**

- When writing commands, variable names, or file paths that a reader
  must type exactly, use code blocks (monospace font) consistently.
  Do not mix inline prose and inline commands.
- When reviewing a reported command error, check for ambiguous
  character substitutions before assuming a logic error. Flag the
  possibility gently — "could this be a 1/l confusion?" — before
  diagnosing further.
- Avoid presenting critical values (MD5 hashes, IP addresses, port
  numbers) in dense paragraph prose. Tables or code blocks are
  preferred — they provide visual separation and reduce
  misreading risk.
- When outputting long hex strings (MD5 hashes), break them into
  labeled rows rather than running them together.

---

## Motor: Intermittent Hand Dexterity

**What it is:** Mir has intermittent hand dexterity issues that
affect typing accuracy and the reliable use of modifier key
combinations. This is not constant — it is unpredictable in onset
and duration. It is difficult to determine where the periodic
paralysis ends and these dexterity issues begin, and vice-versa.

**What this means in practice:**

- Modifier key combinations (Ctrl+C, Ctrl+Alt+T, Shift+click, etc.)
  are physically difficult or unreliable during affected periods.
- Typos in commands are more likely than for a typical user and
  should be interpreted charitably.
- Long unbroken command sequences are harder to execute cleanly.

**Guidelines for contributors:**

- Prefer menu-based or single-key alternatives to multi-key
  shortcuts wherever possible. Document both if both exist.
- Break long command sequences into numbered steps rather than
  chained one-liners. A command that can be split across two lines
  with clear labels is easier to execute accurately than one long
  pipe.
- When a command requires a modifier key (e.g., PowerShell "Run as
  Administrator"), document the menu path as the primary method
  and note the keyboard shortcut as an alternative — not the
  other way around.
- When reviewing a reported command error, consider typo and
  character-substitution explanations before logic explanations.
  Both the visual and motor considerations can contribute to the
  same class of error.

---

## Documentation Style Decisions Driven by Accessibility

Several choices throughout this repository exist specifically because
of the above considerations. They are not stylistic preferences —
they are functional requirements:

| Decision | Reason |
|---|---|
| All commands in code blocks, never inline prose | Visual disambiguation — monospace reduces character confusion |
| Tables for version numbers, IPs, MD5 hashes | Visual separation — dense strings in prose are unreadable |
| Numbered steps rather than chained commands | Motor accommodation — one action at a time is executable |
| KB and GB both shown for file sizes | Visual verification — two formats reduce transcription error |
| Explicit begin/end markers on document blocks | Visual orientation — clear boundaries reduce context loss |
| Menu path documented before keyboard shortcut | Motor accommodation — menus do not require modifier keys |
| Ambiguous characters flagged before execution | Visual accommodation — catch before failure, not after |

---

## How to Work with Mir

If you are collaborating with Mir directly:

- **Typos in her input are real and frequent.** Interpret charitably.
  If a command fails and a character substitution could explain it,
  flag that first.
- **Modifier key suggestions may be declined.** This is not
  preference — it is a physical constraint. Offer menu alternatives.
- **Dense output is hard to read.** When presenting results, use
  tables, labeled sections, or code blocks. Avoid running critical
  values together in paragraph form.
- **Explicit is always better than implicit.** State file names,
  paths, and version numbers explicitly in every relevant context.
  Do not rely on "same as above" or "as before."
- **Context loss is a real risk.** Mir works across multiple sessions
  and systems. Clear document structure and explicit cross-references
  are not overhead — they are the system working as designed.

---

## What This Document Does Not Contain

This document describes working style and communication preferences
only. It does not contain:

- Sensitive medical diagnosis details
- Clinical history
- Medication information
- Any genomic or variant data

For the clinical context behind why this stack was built, see
`06_MEDICAL_CONTEXT.md`.

---

≡≡≡≡≡ END: 05_ACCESSIBILITY_NOTES.md ≡≡≡≡≡  
