# Task list for setup

- Have a public-facing web-page with:
  - Environment setup.
  - How to obtain the repo.
  - How to install the package and developer dependencies in editable mode.
  - All the links required in one place.
  - How to contribute (dev branch target, or just main?).
- Generate some initial issues for people to pick up.
  Ideally we would do the following:
  - 1 group of labels for "complexity"; EG "where to start", "bit more complex", "advanced".
  - 1 group of labels for "topics"; EG "testing", "documentation", "formatting", etc.
  - 1 label that can be assigned to all issues pertaining to the workshop ("Hackathon").
  - 1 label that can be assigned to all PRs pertaining to the workshop.
- FAQs and pitfalls:
  - Don't forget to pull latest changes as development will be happening very fast!
  - Merge conflicts might happen fairly frequently.
  - We need some level of "stop and wait" pertaining to PR reviews... people might be swamped.

We will need Isaac (since it's his codebase) to provide a short overview of what the code does too.
He can provide this to one of the RSEs, or he's welcome to go through it himself on the day.

## Initial Demo

To ensure that we don't break anything, we should add some regression tests prior to letting everyone go wild.
Isaac already has three scripts which we can use for this purpose.
Prior to the workshop, the RSEs should create a branch that turns these scripts (or is heavily inspired by them) into regression tests that are picked up by `pytest`.
(The quality of said test code is not important, it might even want to be deliberately bad so as to encourage / allow for further participant contributions).

At the beginning of the workshop, the "issue -> branch -> develop -> PR -> review -> merge/edit" workflow can then be conveniently showcased, _and_ we guarantee that there are at least some tests in place so that we don't break everything!
