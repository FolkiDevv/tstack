---
name: opus-review
description: Use when the user says "opus review".
---

# opus-review

This is the [review](../review/SKILL.md) skill with Claude Opus 5.5 at high
reasoning effort as both reviewers. Read `review` and follow it, with the
two loops swapped:

- [opus-loop](../opus-loop/SKILL.md) looks for bugs, in place of
  `neon-loop`.
- [opus-security-loop](../opus-security-loop/SKILL.md) looks for security
  holes the change opens, in place of `neon-security-loop`.

Every rule of `review` applies as written. Report each loop under its Opus
name.
