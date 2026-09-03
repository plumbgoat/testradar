---
description: Find code you changed that has no test covering it, then write the missing tests
argument-hint: [optional scope, e.g. a file, module, or directory]
---

Use the testradar skill to find untested code and write the missing tests.

Scope: $ARGUMENTS

If no scope is given, default to what changed in git (uncommitted, staged, or
the last commit), per the skill's instructions. Follow the skill's workflow
exactly: detect the project's test setup, find and rank the gaps, present the
gaps to the user BEFORE writing anything, then write and run tests only for
the gaps the user approves.
