# Writing Unit Tests with Parametrisation

> [!NOTE]
> There are several sub-modules (and functions therein) which could do with having some unit tests added, and which are appropriate for illustrating the ideas that we're going to explore in this issue.
> As such, you may want to explore the codebase yourself, and find other functions that you can add unit tests to.

This issue chooses to focus on writing tests for the `numerics.cumulative_integral` function.
However, we will be describing the steps (and thought process) in a general manner, since the important thing to practice is "thinking about, and coming up with, meaningful tests" just as much as being able to write tests in Python (or `pytest`) syntax.

- [ ] The `numerics.cumulative_integral` file contains only a single function, which performs a "sum of rectangles"-style integral, when given an integrand.
  We will focus on writing some tests for this method.
  - Create a new file, `tests/test_numerics/test_cumulative_integral.py` for your tests to live.
    We will put all our tests for this function inside this file.
  - Examine the `cumulative_integral` function to make sure you understand how it works, and what it is (claiming to!) do.
    Would you be able to compute the analytical value of the function's output by hand, if you were given some simple inputs (EG a straight-line function)?
    Do you notice anything in particular about the call to `jax.numpy.cumsum`?
    What does this mean for certain inputs (think multidimensional arrays)?
  - Write some tests for this function based on the ideas you've come up with above.
    In each test; provide the inputs `y` and `dt` to the function, and then `assert` that the result of calling `cumulative_integral` matches an `expected` value that you will also provide.
    You are not looking to write as many tests as you can think of - you want to write tests that correspond to different use-cases of this function, or different scenarios in which you know _what_ the function should be doing because of the particular forms of the inputs you gave it.
    Remember that - for `cumulative_integral` in particular - you'll want to pick "integrals" that aren't too difficult for you to work out yourself!
    If you're stuck, here are some ideas:
    - What should happen when you integrate the constant function?
    - What should happen when you integrate a linear function?
    - Pick a non-polynomial function and attempt to integrate that.
      Can you compute the answer you expect in a more explicit (but not necessarily as efficient) manner?
      Can you then compare this expectation to what the "faster" call to `jax.numpy` should be doing?
    - What happens for multidimensional arrays?
      Can you check the function is behaving as expected?
      (Hint: try giving a multidimensional array that is a "stack" of 2 of the cases you've worked on already).
    - What happens if you provide `dt` as 0? Or `y` as 0?

## Introducing Parametrisation

By now you probably have several test functions, that look kind-of similar.
In fact, they probably all contain the same steps; just for different values of `y`, `dt`, and `expected`.
It would be easier for us to maintain our code - not to mention add more tests! - if we only needed to write the _instructions_ or _steps_ in the test once, and then just provided a list of input values to try.

This is where _parametrization_ can be helpful; parametrization is essentially us abstracting the test steps away from the test inputs.
It is also part of the reason as to why `pytest`-tests are written as functions; functions take a collection of inputs, do something with them, and spit out an output.
A parametrized test also takes a collection of inputs, does something with them (the test steps), and then spits out a pass/fail result (the `assert` statements).

- [ ] Create a new test function inside the same file you were working in before, that contains all your tests.
  - Have this new test function perform the same steps as your other tests, but take the input values (`y`, `dt`, and `expected` in the case of `cumulative_integral`) to the test as arguments.
  You will likely realise this is a very short function!
  - Next, decorate your test function with [a `pytest.mark.parametrize` marker](https://docs.pytest.org/en/stable/how-to/parametrize.html).
    - The first argument to `pytest.mark.parametrize` should be a tuple of strings, where the strings are the argument names that you'll be providing values for.
    - The second argument to `pytest.mark.parametrize` needs to be a _list containing lists of input values_ - for example `[[y_0, dt_0, expected_0], [y_1, dt_1, expected_1]]`.
      You should be able to copy the values from your previous test cases into this list-of-lists.
  - Double check that `pytest` can now detect all your parametrization cases.
    Also check that they run correctly and they all still pass!
    If everything is working, you can now remove all those individual test functions that are now captured as inputs to your new, parametrized test.
- [ ] (Optional) As you might have figured out, writing a list-of-lists in this way when parametrizing tests isn't super helpful, and can get quite confusing.
  You might also have noticed that the parametrisations themselves are given unique `int` ID numbers, but it's not super easy to tell which is which!
  Fortunately, `pytest` has a solution in the `pytest.param` function.
  In your parametrisation block, replace each set of `[y, dt, expected]` inputs with a `pytest.param(y, dt, expected, id="sensible name")`.
  Can you figure out what the `id` value is doing (and what would be sensible choices for it)?

## Other Functions to Test

If you want a similar task to the above, there are a few other functions in the `numerics` sub-module that we can repeat the above process for.
We've also provided a few hints at test cases to consider, to get you started.

- [ ] `numerics.central_difference`.
  - What happens if the array has a length of exactly 1 (or 2)?
  - How does it handle multidimensional inputs?
- [ ] `numerics.linear_root`.
  - This is just a formula validation but will provide some good testing highlights.
  - Namely it's always doing interpolation, so you can see if it gives the "correct (as in, from what the code is doing)" answer when used on both sensible and non-sensible intervals / functions.
