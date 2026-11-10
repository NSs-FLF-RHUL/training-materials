# DeepSKA Hackathon

Welcome to this hackathon that focuses on testing, and its importance with regards to validating, improving, and documenting code.
The plan for the day is for the morning to be more demonstration-lead; including a recap of the `pytest` framework, the developer workflow (using PRs via GitHub & awaiting review), and a working-together session on one of the starter issues.
We'll also discuss pull-request review, provide some motivation for the day, and give an overview of what the existing code-base is doing.

After the initial demonstration, the remainder of the day will be a "choose your own activity" session, with the "activities" corresponding to the issues that have been opened for you to choose from and work on!
The issues are listed below and have been loosely organised by topic, and by complexity - some issues start simple and then lead on to more complex setups.
You may want to work in pairs or groups if multiple people are interested in tackling the same sorts of problems.
Similarly, you may also be asked to provide review on someone else's pull request if you implemented the previous part, or were working on a related portion of the codebase.

Each task has it's own issue, though some tasks are broken down into several steps.
You will need to make a branch in your personal clone / fork of the `deepSKA` repository to work on your chosen issue ("feature").
On your branch, you will also want to make frequent commits - the steps within each task provide some guidance as to how to break down your work across multiple commits, but in general:

- Commit frequently, but sensibly.
  For example, if you rename a variable across multiple files, it's probably a good idea to make a commit containing the changes to all the files you had to update.
  Other sensible commits would be adding a method to a class, then another commit to add the tests for that method, and then another commit to fix any bugs your tests uncovered in that method!
- Try to only make a commit when the code is _working_.
  This means that if you ever need to "roll back" your changes, you will obtain a working codebase again!
- Leave descriptive commit messages.
  They don't have to be (and shouldn't be) essays, but they should be descriptive enough that you know what you did in them.

> ![NOTE]
> Do not hesitate to ask for help if you get stuck at any point, or have questions about the "how" / "what" / "why" of what we're going today!

## Refactoring / Code Organisation / Object-Oriented Code

The focus on this stream is more on how tests can be leveraged to provide assurances when reorganising or restructuring code.
There will still be points in these tasks where you will have to write tests - but the focal point will be more on using your test coverage (and that which already exists) to ensure that your changes to the codebase do not break anything.
We'll also be touching on / seeing / working with some good design practices, which will be applied to the codebase directly.

### Avoiding "Global" Values in `paper_inputs`

You may notice that in a number of the scripts (and in the regression tests) it is necessary to include an odd-looking `if` statement in the `import` block, which culminates in a `del sys.package` statement.
What is happening is that the "paper inputs" which the scripts need to run (and reproduce the results of the Graber-Cumming-Andersson 2018 & Vela 2016 papers) are imported as "loose constants" from a submodule within the `deepSKA` package.
These loose constants are in fact, not constants - they are more akin to global variables!
As such, if we change their values and re-run the script, we need to have Python "forget" everything it's already loaded before loading it again.

This is not ideal in terms of design - we don't want to be manipulating the package cache anyway, and it would be much better for us to just provide the input values the script needs from a file rather than hard-coding them as loose constants.
As such, in this issue we will be creating a container class for the "paper inputs".
We will set up this class in such a way that _an instance of it_ can be created with the paper inputs, but crucially the codebase itself will not depend on some fixed values it needs to import.
Instead, the code will need to be adapted so that it expects to receive an instance of this class - meaning that the analysis can be run multiple times using different input values _without_ needing to restart Python and clear the package cache.
It also has the added bonus of allowing us to store the input values in separate files, so we have a record of which input file was used.

- [Constructing a Container Class for `paper_inputs`](./refactor/paper-inputs-class.md).
- [Fetch Data Files from Outside the Repository](./refactor/vela-data-fetch.md).
- [Creating a Class for Pinning Tables](./refactor/pinning-tables-class.md).

### Testing and Test-Driven Changes

These tasks will focus on unit testing - not just writing tests, but also organising and designing them.
And, more importantly, why we want them in the first place - if we're going to make later improvements to the code, we need some way of checking that we're not _changing_ what the code is meant to be doing!
We will also explore the idea of negative testing (checking that our code does throw errors if we try and do something silly with it), and motivate the reason for having tests in place before you then undertake expansions or reorganisations of your code.

- [Writing Unit Tests (Parametrisation)](./testing/writing-tests-params.md)
- [Writing Unit Tests (Negative testing)](./testing/negative-testing.md)
- [Writing Unit Tests (Safeguarding against future changes)](./testing/safeguarding.md)
