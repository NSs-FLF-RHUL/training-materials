<https://hackmd.io/7SaiHCTpTLaqK5RoGwxBJg>

# Wed 11th Feb - FLF Training Day

## Setup

Resources going in:

- Had a short analysis script from the participants, that we had looked through prior to the day.
    - We had already identified some issues and created these in the repository.
    - The scripts were already staged as a PR, though this isn't strictly necessary. We could have been developing off the `main` branch if necessary.
- Package infrastructure was already set up, we didn't plan to focus on the actual packaging mechanics.
- We assumed everyone present had a working understanding of Git / GitHub / Python, - though not in a software engineering context - though we did take questions on the more nuianced aspects of these.

## Plan

Our objectives for the training day were - broadly:

- To introduce people to the GitHub development workflow; issues -> branches -> pull request & review -> merging.
    - If there had been more time, we could have setup some examples of resolving conflicts, branch protection rules, and the like.
- To introduce the concept of creating modular code snippets that can be re-used across multiple analyses.
- To introduce the idea of (automated) testing, code formatting, and continuous integration (CI).
- To demonstrate the benefits that testing can provide as the codebase evolves.

## What Actually Happened

### Introduction to the Training Day

Brief overview of what we planned to do, key points being:

- We know you can all code; but coding for research tends to be very output-oriented and doesn't leave much space for reflection, organisation, or maintenance of code.
- What we're going to be doing here is showing you a very different way of thinking and approaching coding. The concepts and ideas are universal, but how they are enacted / enforced across different projects / groups / languages might be different.
- Almost all of what we are going to show you will become more intuitive the more you attempt it! Getting into good habbits takes time, and we're not here to critise you for that. Likewise, questions about the how and why of what we're doing are always welcome - we don't expect this to be immediately intuitive!

### 1: The Development Workflow

We planned to start off with a relatively simple issue, to introduce the development workflow cycle. We chose to do this alongside a relatively common and "cheap" improvement to the script: gathering all the loose parameters into a single structure (`dict` for Python, though `dataclass`es were also mentioned).

The primary purpose of this first step was to demonstrate the development workflow.
- The demonstrator opened the relevant issue for the task that was going to be undertaken. At this point, some of the benefits of actually introducing these changes to the code were also discussed (as well as future generalisations).
- A sub-issue was broken out to show that some pieces of work can be tackled in parts, particularly if there is a lot of work to be done _or_ the work can be done in sensible chunks that don't break the functionality of the rest of the code.
- Importance of creating a branch to work on the feature was then discussed. Avoiding merge conflicts and branch etiquitte were also discussed here, as well as how this workflow is scalable to larger collaborations and asynchronous working.
- The issue was then fixed in a local clone of the repository, on a development branch, with changes introduced in a sequence of sensible commits. Again, discussion of what should go into a single commit was also discussed. Some improvements were deliberately missed, for the purposes of the review later.
- Once the changes had been completed, the branch was pushed to Github and the process of opening a PR and requesting a review was demonstrated. The importance of review was also talked about a lot here.
- The demonstrators then swapped, to show the review process. This includes leaving comments on individual lines, making simple suggestions, opening up discussion threads, and passing these comments back to the author.
- After showing the back-and-forth of the review and adapt process, the PR was merged and the linked issue automatically closed, to complete the demonstration.

### 2: Refactoring Code

The next "core" concept we wanted to demonstrate was the idea of "refactoring": identifying pieces of code that are repeated across the codebase (or, after an abstraction, are repeated) and moving these into a package so they can be recycled.

Key benefits here are that:
- Less code to maintain always better.
- Abstracting code in this manner means that any bugfixes / updates don't have to be propagated to several places in the codebase.
- It opens up the possibility of unit testing for improving the reliability of the code. (This ties into the next portion of the plan).
- For people who aren't familiar with writing packages, this is also a good place to emphasise the purpose of packages. Reusable, abstract code goes into a package. Then, use scripts (or notebooks I guess) for your actual analysis workflows. Other people don't need your scripts (and you may not want them to have them!) but they will most likely have use of the general-purpose, abstract code that you are providing.

Again, this was done adhering to the development workflow.
These benefits were also explained along the way, along with some considerations to make when adapting code from specific to slightly more abstract use.
In the particular example we had, the function that was refactored needed to be adapted to take an additional argument for the parameters (which had previously been global script variables).
Several ways to incorporate the parameters were also discussed (`**kwargs`, a single argument, explicitly listing the needed parameters, etc).

### 3: Unit Testing (and other testing forms)

Now that the package has a "unit", thanks to the previous step, we were ready to discuss testing.
Since this was Python code, the ideas behind writing tests were blended with `pytest`-specific syntax and methods of running.

- The first step was to explain the structure of the `tests` directory, how thet tests were discovered, and how to run them locally. It was also flagged that the tests had been configured to run automatically on PRs, to help reviewers.
- The first test was then written as a simple input/output test. 
    - When testing, use values that make validating the answers easier! You don't need to be using sensible physical values to verify your code is _programmatically_ correct.
- After writing the first input/output test, invite participants to suggest what else should be tested.
    - The idea is to get them thinking about test coverage of edge cases, failure cases, and similar.
    - In this case, another simple input/output test for when the function was called on the boundaries of each of the code-flow-cases was suggested.
    - Other suggestions were checking that an error was thrown when one of the necessary parameters was not available in the `params` dictionary that the function recieved.
- Having written two input/output tests, the idea of parametrising tests was then introduced.
    - Again, this is both a separation of concerns (test workflow vs values to test) and a reduction in code bloat.
- If there is time, an example of negative / failure case testing was also demonstrated.

At an appropriate point, the distinction between regression testing and unit testing was also discussed.
- Unit testing should follow a build-up strategy; test each small component so that you can be confident they're doing their jobs correctly when combined. It also helps identify bugs by isolating the part of the code that's doing the wrong thing. Using realistic or "physical" parameters isn't super important, as you're testing the _programmatic_ correctness of your code.
- Regression / integration tests are more akin to analysis pipelines; they check that when everything is combined it works correctly and produces a meaningful result. Such tests are also conducted by comparing the results to some "known" or "trusted" output, and usually involve more physically-relevant setups.

### 4: Benefits of Testing (codebase evolution)

The idea for this final part was to bring to light the benefits of having put unit tests in place.
We were going to modify the code of the function we refactored and wrote tests for in part 3, with a view to making it more efficient.

- If we did not have any tests, we would have no checks that our changes - which are meant to be purely performant (not functional) - were not introducing bugs _or_ breaking our existing analysis workflows.
- Since we have tests, we now have a basis from which to draw confidence that anything we are changing has not broken the rest of our analysis.
- There are some instances where the tests will have to adapt, particularly if the project decides on an API change, or a reorganisation of the codebase, etc.
- On a related note, it is for this reason that whenever a bug is fixed, a test case is usally added to prevent that bug from coming back into the codebase at a later time!

## Evaluations

### Will

Being provided with a workflow before the workshop was helpful for a number of reasons.
- The primary one being that attendees could come in with some understanding of what the code _was doing_, which helps them understand _why_ the recommendations / changes we were making to the code were helpful.
- It gives a sense of authenticity to the code and the steps being conducted.
- Simply writing down the thought process upon first seeing and reviewing the code was itself a useful exercise and thought experiment to share. It could also be a good "first look exercise" for an actual, fully-fledged workshop.

Sadly, this approach isn't scalable going forward, as you don't really want to be doing prep for the workshop based on code you may not get until a few days prior. However, having a template repository ready to go, with code that already fits the research context - would be a good compromise.

We found that there was a bit too much to get through in the time we had. We'd hoped to get to at least "benefits of testing" to really close the loop on why these development practices really yield benefits, but had to stop at the end of 3. This is partly because testing itself is a very large topic - we could easily do a whole day on how just writing good tests!
