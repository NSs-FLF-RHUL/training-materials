
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
  - Now that `PinningTable` is a `dataclass`, we can implement the `__post_init__` method. This method runs once on our `dataclass`, automatically after an instance is created. This makes it the perfect time to do the one-off expensive calculation of `b_all, delta_v_all, f_all, p_b`.
  - In the `__post_init__` method, assign the `b_all, delta_v_all, f_all` attributes using the `b_knots` method of the instance. Then assign the `p_b` method. The existing code in `deepska.glitch.glitch_rise_model.drag.p_b` should more-or-less directly translate across!
  - Delete the now-unneeded `deepska.glitch.glitch_rise_model.drag.p_b` sub-module.
  - The functions in `deepska.glitch.glitch_rise_model.drag.make_b_profile.py` were using this "global" `p_b` value, which now no longer exists. Edit them appropriately (by making them methods of the `PinningTables` class) so that their functionality is not broken.
  - And finally, update any references to these now-missing functions from `make_b_profile.py`!
