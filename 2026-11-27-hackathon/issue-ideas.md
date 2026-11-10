# Workshop Issue Ideas

## Refactoring / Code Organisation / Object-Oriented Code

Focus mainly on the use of constant imports from the package which is causing the `del sys.package` syntax to be necessary.

- "Paper inputs" should move to a container that the user constructs (`deepska.glitch.data.paper_inputs`).
  - `vela_path` will be handled by someone else, so pretend that's not here right now.
  - This should be wrapped up into a `NamedTuple` class (similar to what `PinningTables` does for the pinning tables).
  - Said class should have a `classmethod` that creates an instance with the current hard-coded values.
  - Modules that were previously importing these values and using them "raw" will need to be properly parametrised with the new object.
  - Note that the `Spin` class shares 5 of the fields that the proposed new class has too! So the new class should have a `spin` attribute that corresponds to this attribute!
  - (Optional) allow `PaperInputs` to be read from a configuration file.
    - Create a `classmethod` on the `PaperInputs` class, `.from_file`.
- The Vela residuals should not be tracked in the repo. Let's store them in another file and then have a function to fetch them if necessary.
  - We can move the residuals to GH releases to fetch them from the web.
  - All the interfaces then take an array directly, not the path.
  - Some functions can be moved around to accomodate loading; `vela_data` for example can just be part of this new `fetch_vela_residuals` function.
  - (Optional) We can take this further, and turn the `Vela(NamedTuple)` in `kinds.py` into a proper class with attached methods.
    - Move the `Vela` class definition into it's own file. Somewhere like `glitch.data.vela` would be appropriate.
    - The "loader" function that you just made can become a `classmethod` of the `Vela` class - it can also return a `Vela` instance.
    - Any "intermediate" functions you find that are involved in loading the Vela residuals / data should also be moved into your new file. You can decide (or ask for advice on) whether they should become methods, classmethods, or staticmethods. However you choose to approach things, don't try and move too many things at once! Make small, incremental commits and make sure the regression tests pass each time, before you move on to the next change!
    - From a pre-emptive sweep of the repository; the following functions will need to be moved around: `paper_vela` (which will become that class method), `vela_data`, and `vela_from_glitch`.
    - Don't forget to "clean up" / delete files that are now un-necessary once you reorganise things!

The remaining ideas here revolve around turning the `PinningTables` class into something more extensive, by attaching other functions from inside the package "to" this class to make the codebase more object-oriented, _and_ importantly a lot more parametrisable.
It also is necessary if we want to continue addressing the `sys.package` remark that we noted right at the start.
Note that whilst you're conducting these tasks, you might also want to consider whether you can come up with more descriptive names for the methods / functions that you're refactoring (EG can you avoid single-letter or notation-based names like `b`, etc)?

- Pinning tables (`deepska.glitch.data.b_data`):
  - The definition of `PinningTables` can be moved from `kinds.py` into this submodule, since it's only used here.
  - `PinningTables` should be given a docstring that describes the attributes.
  - `PinningTables` should have a `classmethod` that recreates the pinning table used in the paper, IE the current hard-coded values.
  - Note that modules that were previously importing the individual constants will now need to be properly parametrised to take a `PinningTables` object.
- The `b_knots` function (`deepska.glitch.glitch_rise_model.drag.b`) accepts a `PinningTables` as it's only input. It might as well be a method on the class, and not a separate function.
  - Move `b`, `b_knots`, `over_cases`, and `over_knots` into the `b_data.py` file. If it hasn't been done so already, also move the `PinningTables` class definition into this file (be careful not to conflict with the above issue!).
  - Convert `b_knots` to be a method on the `PinningTables` class. Note that this will require you to change the syntax being used in the `p_b.py` file, where `b_knots` is being called (though we'll come to that in a second...!).
  - Rename the `b` function to `_b`, to mark it as private. Make it a `staticmethod` of the `PinningTables` class.
  - Decorate the `_b` method such that the vmap-ing, that is applied by `over_cases` and `over_knots` currently, is applied using the `functools.partial` decorator.
    You will need to stack `functools.partial` on top of another `functools.partial`, then have `staticmethod` on top.
    Essentially;

    ```python
    class PinningTables:

      @staticmethod
      @functools.partial(jax.vmap, ) # Replicates `over_cases`
      @functools.partial(jax.vmap, ) # Replicates `over_knots`
      def _b(...):
        ...

    ```

    (There's also the `every_pair` function for inspiration. In fact, this could become a decorator...)

  - The `b_knots` method should now be able to `return self._b(knots, cases, self.delta)` of the respective arguments.
- The `deepska.glitch.glitch_rise_model.drag.p_b` file exposes 4 values at module-level; `b_all, delta_v_all, f_all, p_b`. These are, however, all directly computed from a `PinningTables` instance!
  - Reading the comments in this file, you should notice that this is an expensive computation. We don't want to do this more than once. Therefore, we should compute these values when a `PinningTables` instance is constructed, that is, within the `__init__` method.
  - We are now adding "derived attributes" to our `PinningTables` class, which will not be supported by `NamedTuple`. As such, convert `PinningTables` from a `NamedTuple` to [a `dataclass`](https://docs.python.org/3/library/dataclasses.html). Take care when doing this - changing to a `dataclass` will change how a `PinningTable` instance is created, so your earlier function that creates a `PinningTable` with the paper inputs might need to be updated too.
  - Now that `PinningTable` is a `dataclass`, we can implement the `__post_init__` method. This method runs once on our dataclass, automatically after an instance is created. This makes it the perfect time to do the one-off expensive calculation of `b_all, delta_v_all, f_all, p_b`.
  - In the `__post_init__` method, assign the `b_all, delta_v_all, f_all` attributes using the `b_knots` method of the instance. Then assign the `p_b` method. The existing code in `deepska.glitch.glitch_rise_model.drag.p_b` should more-or-less directly translate across!
  - Delete the now-unneeded `deepska.glitch.glitch_rise_model.drag.p_b` sub-module.
  - The functions in `deepska.glitch.glitch_rise_model.drag.make_b_profile.py` were using this "global" `p_b` value, which now no longer exists. Edit them appropriately (by making them methods of the `PinningTables` class) so that their functionality is not broken.
  - And finally, update any references to these now-missing functions from `make_b_profile.py`!

## Testing and Code Improvements

Focus mainly on writing tests, but also organising and designing them. And, more importantly, why we want them in the first place - if we're going to make later improvements to the code, we need some way of checking that we're not _changing_ what the code is meant to be doing!

- The `numerics.cumulative_integral` file contains only a single function, which performs a "sum of rectangles" -style integral, when given an integrand. We will focus on writing some tests for this method.
  - Create a new file, `tests/test_numerics/test_cumulative_integral.py` for your tests to live. We will put all our tests for this function inside this file.
  - Examine the `cumulative_integral` function to make sure you understand how it works. Do you notice anything in particular about the call to `jax.numpy.cumsum`? What does this mean for certain inputs (think multidimensional arrays)?
  - Write some tests for this function based on the ideas you've come up with above. In each test; provide the inputs `y` and `dt` to the function, and then `assert` that the result of calling `cumulative_integral` matches an `expected` value that you will also provide. Remember that you want to pick "integrals" to perform that you can compute the answer (by which we mean the result the function will spit out) of! If you're stuck, here are some ideas:
    - What should happen when you integrate the constant function?
    - What should happen when you integrate a linear function?
    - Pick a non-polynomial function and attempt to integrate that. Can you compute the answer you expect in a more explicit (but not necessarily as efficient) manner? Can you then compare this expectation to what the "faster" call to `jax.numpy` should be doing?
    - What happens for multidimensional arrays? Can you check the function is behaving as expected? (Hint: try giving a multidimensional array that is a "stack" of 2 of the cases you've worked on already).
    - What happens if you provide `dt` as 0? Or `y` as 0?
- This continues the `numerics.cumulative_integral` task above. By now you probably have several test functions, that look kind-of similar. In fact, they probably all contain the same steps, just for different values of `y`, `dt`, and `expected`. It would be easier for us to maintain our code - not to mention add more tests! - if we only needed to write the _instructions_ or _steps_ in the test once, and then just provided a list of input values to try. This is where _parametrization_ can be helpful.
  - Create a new test function inside the same file, that performs the same steps as your other tests, but takes the input values to the test as arguments. You will likely realise this is a very short function!
  - Next, decorate your test function with [a `pytest.mark.parametrize` marker](https://docs.pytest.org/en/stable/how-to/parametrize.html). The first argument to `pytest.mark.parametrize` should be a tuple of strings, where the strings are the argument names that you'll be providing values for. The second argument to `pytest.mark.parametrize` needs to be a _list containing lists of input values_ - for example `[[y_0, dt_0, expected_0], [y_1, dt_1, expected_1]]`. You should be able to copy the values from your previous test cases into this list-of-lists.
  - Double check that `pytest` can now detect all your parametrization cases. If it can, you can remove the individual test functions you were using before!
  - (Optional) As you might have figured out, writing a list-of-lists in this way isn't super helpful. You might also have noticed that the parametrisations themselves are given unique `int` ID numbers, but it's not super easy to tell which is which! Fortunately, `pytest` has a solution in the `pytest.param` function. In your parametrisation block, replace each set of `[y, dt, expected]` inputs with a `pytest.param(y, dt, expected, id="sensible name")`. Can you figure out what the `id` value is doing (and what would be sensible choices for it)?
- If people want a similar task to the above, there are a few other functions in the numerics package that we can repeat the above process for.
  - `numerics.central_difference`; what happens if the array has exactly length 1 or 2? How does it handle multidimensional inputs?
  - `numerics.linear_root`; this is just a formula validation but will provide some good testing highlights. Namely it's always doing interpolation so you can see if it gives the "correct" answer when used on silly intervals.

<!-- Negative testing -->
- (Slightly more complex code, but the ideas are the same as the `cumulative_integral` issue). The `numerics.every_pair` file contains the `every_pair` function. Read this function to understand what it does. Then, create a suitable suite of tests for it, that cover the important ways in which it should behave.
  - Hint: `every_pair` takes another function as an input - it would be wise to pick function(s) that use their arguments in simple ways, rather than going for something complicated.
  - Hint: if you are stuggling to figure out what the inputs to each test case should be, they will be something like: a function `f` that takes 2 (or more) arguments (to be given as the `run` argument to `every_pair`), two arrays corresponding to the values that need to be looped over (`inner`, `outer`), and another input `args` that contains any values that are held constant over the runs.
  - Once you have a few test cases, try to parametrise your various test functions in the manner described above.
  - Negative testing: you might have spotted that `every_pair` relies on the user providing `args` correctly, based on the function `run` that is given. What do you expect to happen if, for example, `run` doesn't take any extra values, but some `args` are passed anyway? Can you think of a way to write a test to see if what you expect does indeed happen? [Hint: `pytest` might be able to assist...](https://docs.pytest.org/en/stable/how-to/assert.html#assertraises).

<!-- Testing as a means of validating refactoring-->
- (Slightly more complex code, but ideas are the same as the `cumulative_integral` issue). The `numerics.invert` file contains code that implements the bisection method to find roots of a function. Read this function to understand what it does, note that it also relies on two further sub-functions to operate correctly.
  - Once you understand what the `invert`, `bisection`, and `bis` methods are doing, write a suitable suite of tests for each of them.
  - (Optional) make clarifying improvements to the docstrings of these functions as you write tests and begin to understand what they're doing. For example, the description of `param f` in the `invert` function can be re-worded to "function to invert, must be monotonic on the interval `(low, high)`".
- The `target` parameter passed to `invert` specifies the function value we are looking for the image of (`f(x) = target` means we're looking for `x`). However, this problem can be solved equivalently by looking for the value `x` where `g(x) = 0`, with `g(x) = g(x) = f(x) - target`, which reduces the number of parameters we need in our functions (as well as the complexity of root-finding in those functions).
  - Modify the `invert` function so that it constructs the function `g` from the function `f` and `target`, using an inline function definition (either `lambda` or explicit `def` statement).
  - Next, modify the `bisection` and `bis` functions so that they no longer take the `target` argument - they should operate as if they are always attempting to solve `f(x) = 0`.
  - Finally, pass the `g` function constructed in `invert` to `bisection`, rather than `f`.
  - Throughout this process, you may also need to adapt the docstrings of these functions.
  - As you are making your changes, examine any existing tests that are in place for these functions. Without removing any test cases, update the tests appropriately so they use the new function signatures, _but are still covering the same cases as before_!
- The `up` parameter that is computed inside `invert` and then passed to `bisection` and `bis` is actually un-necessary once we swap to using the function `g(x) = f(x) - target` in the previous step. We do not actually need to know whether the function is strictly increasing (going "up") or decreasing ("down") to implement the bisection step. Instead, we can compute the midpoint of the interval, and examine the sign of the product of `g` evaluated at the midpoint.
  - Modify the `invert` (and `bisection` & `bis` functions) so that they no longer take the `up` argument. Instead; to determine which half of the bisected interval `(low, high)` to keep, evaluate `g(mid)*g(low)`. If this product is negative (or zero), then the root of `g` must lie within `(low, mid)`. Otherwise, the root must lie within `(mid, high)`.
  - Check that your existing tests still pass after you make this modification (and any necessary edits to how the functions are called in the tests).
