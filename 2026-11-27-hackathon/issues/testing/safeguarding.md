# Testing for Future-Proofing

Implementing tests helps us both validate that our code is working as we intend it _right now_, and also serves as a guard for us when we update, edit, or extend our code in the future.
Having tests already in place is important when we continue development, so we can be alerted if a change we've made breaks something we were previously depending on, or is otherwise incompatible.
Without such tests, we run the risk of breaking our previous analysis scripts, which in turn means we have lost the ability to reproduce our results.
If you are releasing code to the public, ensuring you are backwards-compatible is also important since your user-base will typically always be lagging behind your most up-to-date code state (and may want to be able to re-create any results you have "claimed" to produce with said code-base).

In this task, we'll be looking at the `numerics.invert` file.
This file contains an implementation of the bisection algorithm via three (nested) functions; the `invert` function which is the top-level scope and the function that is exposed to the rest of the package, the `bisection` function which actually runs the bisection algorithm, and the `bis` method that performs a single step of the bisection algorithm.

> [!NOTE]
> If you are not familiar with the use of [the Bisection algorithm](https://en.wikipedia.org/wiki/Bisection_method) to find roots of functions, ask a helper (or Isaac if he's free).

## Create some "Normal" Unit Tests

- [ ] Using the same ideas as the [writing parametrised tests](./writing-tests-params.md) task, write a suite of tests for the three functions inside the `numerics.invert` file.
  - (Optional) You can try and parametrise your tests, if you want to.
  - In the interest of time, don't go too overboard with test cases.
    If you have one test case for each of the following, you should have a sufficient number of tests to proceed.
    - A test confirming that `bis` correctly identifies the half-interval to preserve, in each of the following cases: the function is increasing (`up` is `True`) and the root is in the lower-half interval, the function is decreasing and the root is in the lower-half interval, the function is increasing and the root is in the upper-half interval, and the function is decreasing and the root is in the upper-half interval.
    - At least one test that checks what happens when `bis` does a bisection, and the root is _exactly_ at the midpoint.
    - `bisection`; at least two tests, varying the number of bisections performed, and confirming that the resulting interval matches the interval we'd expect to obtain after the given number of bisections.
    You could also check both of the `True/False` cases for the `up` argument.
    - `invert`; choose a couple of functions (either piecewise-linear or periodic) and check that - given the start and end intervals you provide - the result obtained is the linear-root approximation based on the final "feasible interval" that you expect to be returned from `bisection`.
- [ ] (Optional) As you are writing the above tests, you might find it helpful to expand or clarify some of the docstrings that support these functions.
  For example, the description of `param f` in the `invert` function can be re-worded to "function to invert, must be monotonic on the interval `(low, high)`".

## Checking for Backwards Compatibility

Now that we have some tests in place for our function, we are going to pretend we're coming back to our code after a few months and want to make some improvements to our implementation in the `numerics.invert` file.

There are two improvements that we can make.

- [ ] The `target` parameter passed to `invert` specifies the function value we are looking for the image of (`f(x) = target` means `invert` will return `x`).
  However, this problem can be solved equivalently by looking for the value `x` where `g(x) = 0`, with `g(x) = g(x) = f(x) - target`, which reduces the number of parameters we need in our functions (as well as the complexity of root-finding in those functions).
  - Modify the `invert` function so that it constructs the function `g` from the function `f` and `target`, using an inline function definition (either `lambda` or explicit `def` statement).
  - Next, modify the `bisection` and `bis` functions so that they no longer take the `target` argument - they should operate as if they are always attempting to solve `f(x) = 0`.
  - Finally, pass the `g` function constructed in `invert` to `bisection`, rather than `f`.
  - Throughout this process, you may also need to adapt the docstrings of these functions.
  - As you are making your changes, examine any existing tests that are in place for these functions.
    Without removing any test cases, update the tests appropriately so they use the new function signatures (and in the case of `bisection` and `bis`, they are now expecting to always look for `target = 0`), _but are still covering the same cases as before_!
    Make sure that, even after your changes, the tests are still passing.
- [ ] The `up` parameter that is computed inside `invert`, and then passed to `bisection` and `bis` is actually un-necessary once we swap to using the function `g(x) = f(x) - target` in the previous step.
  We do not actually need to know whether the function is increasing (going "up") or decreasing ("down") to implement the bisection step.
  Instead, we can compute the midpoint of the interval, and examine the sign of the product of `g(mid)` with `g(low)` (or with `g(high)`).
  - Modify the `bisection` & `bis` functions so that they no longer take the `up` argument.
    Instead; to determine which half of the bisected interval `(low, high)` to keep, evaluate `g(mid)*g(low)` (or `g(mid)*g(high)`).
    If this product is negative (or zero), then the root of `g` must lie within `(low, mid)` (respectively `(mid,high)`).
    Note: depending on how `bis` handles the case when the `mid`-point is _exactly_ at the root, choosing to evaluate one product over the other will result in you having a slightly easier implementation.
  - Check that your existing tests still pass after you make this modification (and any necessary edits to how the functions are called in the tests).
