# Creating a Container Class for `paper_inputs`

The loose imports come from the `deepska.glitch.data.paper_inputs` submodule.
We will wrap these values up into a "container class" (similar to the ones that can be found in `kinds.py`).
We will also equip this class with a class-method that can create an instance that contains the paper input values, but in general we will allow instances of this class to be created with arbitrary values, and the codebase will be prepared as such.

- [ ] Create a container class inside `deepska.glitch.data.paper_inputs`.
  Choose a suitable name for your class (`PaperInputs` will suffice if nothing more appropriate comes to mind, which is the name we'll use going forwards).
  - Given that this class is going to exist solely as a record of values, it might be appropriate to make use of some of Python's infrastructure.
    You could consider sub-classing from `NamedTuple` (similar to what the classes in `kinds.py` do), or [using Python's `dataclass`es](https://docs.python.org/3/library/dataclasses.html).
    Our recommendation would be to use `dataclass`, but you can get it to work with either.
  - Importantly, a user should be able to create an instance of your `PaperInputs` class in the usual way; by specifying a value for each of the attributes.
    If you have used `NamedTuple` or `dataclass`, you can avoid actually writing the `__init__` method yourself.
  - Do not include the `vela_path` as an attribute - this is being handled [by another issue](FIXME).
    Pretend it doesn't exist for now!
- [ ] Add a `classmethod` (call it `vela_2016_defaults` if you can't think of a better name) to your `PaperInputs` class which creates (and returns) an instance of the class whose attributes are set to the current hard-coded values found in the file.
- [ ] Our next objective is to go through the codebase and replace all of the "loose-value" imports with references to our new container class.
  - Identify the files in the codebase that `import` from `paper_inputs` (using `CTRL+F`, or your IDE's "search files" feature might help you here)!
  - For each file, make the necessary changes:
    - Any functions that are using the hard-coded values should instead take a `PaperInputs` instance as an argument, and use the value of the corresponding attribute instead of the loose-value that's being imported.
    - The scripts themselves will need to make use of your `vela_2016_defaults` class-method to create the appropriate instance of the class, and then pass it to the sub-functions they call as appropriate.
    - The only thing that should be `import`ed from `paper_inputs` in these files should be your `PaperInputs` class, once you're done with your changes.

Once you have implemented the necessary changes on your branch, push it to GitHub and open a pull-request.
Describe what you've done, and link your pull-request to this issue.
Then, find someone to be your reviewer!

## Stretch Goals

If you want to see how to develop these ideas further, here are a couple of extension tasks.
You might want to tackle these in a new pull-request, which builds on top of the one you created for the tasks above.

> [!NOTE]
> Some of these tasks may introduce conflicts with the others - in which case you should talk to your fellow developers and make sure that you are both aware of what the other is working on, and the changes they plan to implement.
> You might want to discuss the approach you want to take with your fellow developers, so you can both anticipate some of the problems you'll encounter and adapt your approaches to avoid conflicts.

- [ ] You might notice that the `Spin` class, in `kinds.py`, shares 5 of the fields that the `PaperInputs` class has!
  (`omega0`, `dom_crit`, `dt`, `n_t`, `profile_steps`).
  This is somewhat wasteful on our part - we should have `PaperInputs` create and store the corresponding `Spin` object, rather than storing the 5 attributes separately as it's currently doing.
  We should remove the 5 attributes that are shared with the `Spin` class from `PaperInputs`, and replace them with a single `spin` attribute, which will store the corresponding `Spin` instance.
  - At this point, you will need to make a decision on how instances of `PaperInputs` should be created.
    Do you want to make the user pass the 5 attributes explicitly, EG `PaperInputs(..., omega0, dom_crit, ...)`, and then create a `Spin` instance inside the `__init__` method to assign to `.spin`?
    Do you want to make the user pass a `Spin` instance to the constructor, EG `PaperInputs(..., Spin(omega0, dom_crit, ...), ...)`, and then assign this `Spin` instance to `.spin` directly?
    Do you want to allow both?
    You decision to pick one or the other (or the combination) is likely to be influenced by the existing structure of the `PaperInputs` class (and in particular whether it is a `NamedTuple` or `dataclass`).
  - Once you have decided on your chosen redesign, be sure to adapt the necessary parts of the codebase!
    Note that this may include your `vela_2016_defaults` class-method from the previous issue list, above.
- [ ] By now you will also have noticed that `PaperInputs` takes _a lot_ of input arguments - almost so many that it's not really feasible to explicitly type out the command to create an instance of the class from them.
    As such, we should consider allowing the values to be read from a configuration file.
  - Design an appropriate configuration file format for the `PaperInputs` class.
    A `JSON` file would be our recommendation, if you are unsure of where to start - a `JSON` file is essentially just a list of keys and fields (in Python, dictionaries are direct representations of `JSON` files).
  - Create a `classmethod` on the `PaperInputs` class, `.from_file`, that can read a file of the format you've designed above and create the corresponding `PaperInputs` instance.
    The method will likely look something like the following:

    ```python
    class PaperInputs:

      @classmethod
      def from_file(cls, file_path: Path) -> PaperInputs:
        file_data = ... # load your file using a Python library, like json

        # If necessary, manipulate values read in from the file to conform
        # to the Python types that PaperInputs is expecting. For example,
        # b_core_range will likely be read in as a list, but it needs to
        # be converted to a `jax.Array` object
        file_data["b_cores"] = jnp.array(...)
        ...

        # Having read the file data, now return a class instance created
        # from it
        return cls(
          dr = file_data["dr"],
          n_tov = file_data["n_tov"],
          ... # etc
        )
    ```

- [ ] Once `PaperInputs` can be created from a file, there isn't really much need for us to store the hard-coded values in the `vela_2016_inputs` class-method any more.
  Instead, we should create a data-file that corresponds to these inputs, and put this data-file in a publicaly accessible place - see what we're doing with the Vela residuals for inspiration on how to do this.
  The `vela_2016_inputs` function can then just fetch this data-file from the website, and call our `from_file` method with the result!
  Note that this will require `from_file` being able to accept either a file-path, _or_ a file-pointer object as an input.
