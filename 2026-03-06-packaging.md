
# Fri 6th March - Online 2hr Session

## Setup

We had a refactored script that had been worked on from a previous workshop day.
we had deliberately left the code in a state where improvements could be made - largely through use of the `numpy` library.
Importantly, the code was already inside the package, and had appropriate test coverage so as to capture any mistakes that could be made in the changes to follow.

The package configuration itself was up to date.
This is important because we also wanted to flag the various differences between dependency types and how one actually adds dependencies to packages in such a way that `pip` / `conda` / etc recognise.

## Plan

Objectives were broadly:

- Set the scene.
  - In particular it's important to emphasise that code development happens in stages.
    It is normal to go through several refactoring + efficiency improvements from what starts as a research script.
  - In this case, we have identified places where an external library may help us avoid "re-inventing the wheel" and / or perform more efficient operations than us doing it ourselves.
- Cover considerations when looking for libraries to depend on, what it means to be a dependency, and how to update the metadata in your package to reflect this.
- Cover the process of updating the package functionality with that from the new library.
  As a bonus, this also covers some `numpy`-specific efficiency / speedup syntax.

## 01 - Considerations for External Libraries

First of all - how do you find candidate libraries to use to avoid re-writing code someone else has already written?

- Google search & look at the READMEs / GitHub pages of results.
- Ask within your research group - you will likely get recommendations that are more relevant to your field as a bonus.
- Ask the RSEs.
- Ask in online forums.

Adding a dependency to your package comes with an important consequence: your code is now tied to that dependency - it won't work without it!
Furthermore, anyone else who wants to use your package will also need to fetch this dependency and install it on their machine too, alongside your package.
If your dependency breaks, well good luck running your own package code!

As such, there are a number of general things to keep in mind when looking for an external library that (may) provide functionality you are looking for in your package.

- Is the library well-tested?
  - You don't want to be chasing bugs in your code only to find out it's someone else's code that is the issue!
  - Test coverage also gives some assurances against the dependency breaking.
- Is it well-maintained / regularly updated?
  - "Abandoned" libraries make poor dependencies - a time will come when the core software moves on but this library does not, which can cause incompatibility issues.
    It can also cause you to miss out on newer features that are added to the core language, or be stuck with unsupported versions of a language in the event of major version updates.
  - You can usually check for signs of life / activity on a project by examining it's GitHub repository (if it's open source).
    Look at how many open issues there are, how much engagement on said issues and pull requests there are, and how frequent the commit history and release history is.
- How big or "bloated" is the library?
  - You want to select your dependencies sensibly - they should provide the functionality you need, but shouldn't provide an excess of features you don't need.
  - As such, you want to be as minimalist as possible.
    If you just need a framework for working with arrays, there's no need to pull in all of `tensorflow` or `pytorch`.
    Doing so will result in your package install time increasing, as well as the memory space for your package being excessive.
    It also makes you vulnerable to problems in those packages that are unrelated to your work - if you're not using `pytorch`'s machine-learning functionality, but suddenly `pytorch` goes down due to a bug in that part of the code, your work is also broken despite not directly using the broken feature!
  - More complex libraries can also require increasingly complex installation procedures, which is quite off-putting for users.
    A pure-Python researcher typically doesn't want to have to learn about compilers to install your package, for example.
- Is the dependency well-documented?
  - And related: is it easy to use?
- Do any of your **existing** dependencies suffice?
  - Remember: fewer dependencies are generally better!

Particularly in research, there may be no single library that does exactly what you want.
In such cases, it is still a good idea to re-use what you can from other libraries.
If they are open to external contributions, it is also a much better idea to contribute the functionality that you need **directly to** those libraries rather than trying to implement it from scratch yourself, in your own work.

## 02 - Technicalities about Dependencies

This is covered primarily in the context of Python packages, however there are analogous terms for other languages.
However, keep in mind that the install process will be different for other languages too: compiled languages will typically need to build the library object and create links to it, whereas a Python package install is typically just copying files to a location on the computer and telling Python where that is.

Dependencies can actually be one of three things:

- Build dependencies
- Host dependencies
- Runtime dependencies

(If you have ever tried to upload a package to conda-forge, you'll likely be familiar with these terms).

If you're only working in Python and writing pure-Python packages, you'll typically only ever need to be concerned with **runtime dependencies**.
Runtime dependencies are libraries / packages that need to be installed on the computer when your package code is being used / run - if you `import numpy as np` in your package code, then `numpy` is a runtime dependency.
For a Python project, runtime dependencies are specified in the `dependencies` field of the `pyproject.toml` file.
When your package is installed, the libraries listed in this entry will also be installed alongside it.
This means that you are effectively forcing a user to install all libraries you list as dependencies of your package, when they install your package (though `pip`, `conda`, `uv`, etc handle automatically fetching dependencies for you).

**Build dependencies** are libraries or tools that are required to build / install your package.
They need to be present for the install process, but can be removed (and will be removed!) afterwards.
For Python projects, they are specified in the `build-system` entry in `pyproject.toml` - typically you only ever see this field populated with `setuptools` or whichever wheel-building package the project maintainer has selected.
However more complex packages like `numpy` specify things like C compilers in this entry, since they build and link to C libraries as part of installing `numpy` - once the install is done however, these compilers are no longer needed and so can be removed.

**Host dependencies** you typically never have to worry about.
They only matter when you are building a package on a different machine / in a different environment to the machine / environment you want to install into, which should hopefully be never!

Python (and other languages) also allow for optional (runtime) dependencies to be installed alongside the main package.
Most projects will provide some combination of optional `dev`(eloper), `test`, and `doc`(umentation) dependencies, which are additional packages not required for users to run the code, but that are required when developing or adding features to it.
`pyproject.toml` files have an `optional-dependencies` entry which handles this.
Other languages have alternative ways of handling such things (like workspaces in Julia, for example).
