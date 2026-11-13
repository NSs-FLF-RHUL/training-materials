# Avoiding Tracking Large Data Files

The Vela-paper residuals [are stored in a `.csv` file](https://github.com/NSs-FLF-RHUL/deepSKA/blob/Isaac_test/src/deepska/glitch/data/palfreyman2018_vela_glitch_residuals.csv)  that is currently tracked by the repository.
This isn't ideal, since this file isn't actually part of the code and is "bloating" the repository with it's 46,000-ish lines of pure numerical data.
Furthermore, it is _just one data file_ that, whilst we need it to recreate our analysis, might not be the only residual file we ever want to use in our analysis.

What we want to do is remove the residual data from the repository, but still have a means of accessing it when we run the scripts that produce our paper results.
The solution, therefore, is to host the data somewhere it can be publicly fetched (we'll use GitHub releases for this) and then fetch it when we need it in the script.
For the sake of reproducibility (and our own convenience) we'll also add a function to the package that fetches this dataset from the web.

- [ ] Create [a new GitHub (pre-)release for the `deepSKA` package](https://github.com/NSs-FLF-RHUL/deepSKA/releases), _or_ edit the [existing "Regression Outputs for..."](https://github.com/NSs-FLF-RHUL/deepSKA/releases/tag/glitchsim-regression-outputs) release so that the Vela residuals file is attached to the `assets` of the release.
  - Take note of the download link for this file, we will need it later!
  - Note that you won't have to open a pull-request to do this step, it will be done entirely through the web-interface, since you're not actually changing the code (you're changing the repository metadata).
- [ ] Next, we're going to write the replacement function that is going to load the Vela data from the web, rather than expecting it to be a local file.
  - The `deepska.glitch.data.paper_vela` submodule contains the function responsible for loading the Vela data from the `vela_path` global variable in, which in `deepska.glitch.data.paper_inputs`. However, this method also makes calls to two further sub-functions, `vela_data` and `vela_up_to`, which in turn may call further sub-functions.
  - [ ] Your first objective should be to gather all the "single-use" subfunctions into the `deepska.glitch.data.paper_vela` submodule itself. A "single-use" function is a function that is defined in it's own submodule, but is only ever used once by the package in another submodule. The `vela_data` and `vela_up_to` functions are two such examples of this. Once you move a "single-use" function into the `deepska.glitch.data.paper_vela` submodule, you can of course remove the now-unnecessary submodule. We recommend doing each of these "moves" in their own commit, to make sure that nothing breaks as you reduce the number of submodules in the package and gather related code together.
    - If you are struggling to find all the "single-use" subfunctions, here is a list of the ones that we spotted when creating this issue:
      - `deepska.glitch.data.vela_from_glitch`
      - `deepska.glitch.data.vela_data`
      - `deepska.glitch.data.bin_series` <- though you might want to leave that one where it is, since it's not Vela-specific.
      - `deepska.glitch.data.vela_up_to`
  - [ ] Now that everything we need to load the Vela data is in one file, modify the `paper_vela` function so that it can fetch the dataset either from the web, or from a local-file.

    ```python
    def paper_vela(t_max: float, *, source: Path | str | None = None) -> Vela:
      ...
    ```

    If `source` is provided, the `paper_vela` function should operate exactly how it currently does: by loading the file given at the `source` location.
    If `source` is `None`, then have the function fetch the Vela residuals from the web, using the file you attached to the GitHub release.
    You will need to make use of [the `requests` package](https://requests.readthedocs.io/en/latest/) to download the file from the internet (and add it to the `deepSKA` dependency list, if it's not already there).
    You will also need to use `io.StringIO` to convert the [downloaded content](https://requests.readthedocs.io/en/latest/api/#requests.Response.content) into a file-string that can be loaded.
    In any case (`source` is a file-path, file-name, or `io.StringIO` object) you should then be able to use `np.loadtxt` to load the data.
    - Note that once you have edited the `paper_vela` function, you'll also need to edit the scripts so that they default to fetching the data from the internet!

## (Optional) A Residuals Class

The `paper_vela` function returns an instance of the `Vela` class, which is essentially a container that holds information about residuals.
This format would be applicable to other datasets in the same format at the Vela-dataset, and so it makes sense to consider creating a class for representing such data, which will be a generalisation of the existing `Vela` class.

The existing `Vela` class - found in `deepska.glitch.kinds` - is currently just a `NamedTuple` container-class for this information.

- [ ] First, let's do some more reorganising.
  Move the `Vela` class into the `paper_vela` submodule, and be sure to update any references that need to change due to the class changing location.
  If you're using PyCharm or VSCode, these IDEs might even be able to do this for you.
  If you're not though, just do a sweep through the codebase for references that match `Vela`, and change any files that were previously importing from `kinds.` to import from `paper_vela` instead.
- [ ] Now, we're free to make our more general `Residuals` class... though what we're actually going to do is just rename the `Vela` class to `Residuals`, and add some functionality to it!
  - As always with a renaming, you'll need to update any references that were using the old name to the new name.
    PyCharm and VSCode both have a "rename symbol" option that appears if you right-click on the class name - this will automatically handle the renaming process across all the files!
  - You may also want to give the `Residuals` class a proper docstring, and re-name some of the attributes to be more descriptive.
    For example, `vela_bin` -> `bin_mean_residual`.
    The docstring should also describe the attributes, rather than using in-line comments next to their declaration.
  - You may also want to consider swapping from a `NamedTuple` to a `dataclass`, so you can pass the attribute-values to `__init__` via keywords, rather than depending on the order.
- [ ] Finally, we're going to convert `paper_vela` (and the sub-functions therein) to class methods of the `Residuals` class.
  - Give `Residuals` a classmethod, called `from_timing_residuals`.
    `from_timing_residuals` should just be a re-skin of the existing `paper_vela` method, except it is now a (class)method on the `Residuals` class, and returns a `Residuals` instance rather than a `Vela` instance.

    ```python
    class Residuals:
      ...

      @classmethod
      def from_timing_residuals(cls, t_max: float, *, source: Path | str | None = None) -> "Residuals":
        ...
    ```

    Don't forget to update any references to `paper_vela` to use the new method you've created instead, and be sure to mention in the `from_timing_residuals` docstring that it will default to fetching the Vela-dataset if not told otherwise!
  - You now have a choice as to what to do with the sub-functions that `paper_vela` made use of.
    They should all - at a minimum - be given more descriptive names to fit the general setting (EG `vela_data` -> `read_and_bin_residuals`).
    You might decide to make them private methods (prepend an `_` to the names) if you want to prevent users from directly interacting with these sub-functions (implying they should always load `Residuals` using the classmethod you just wrote).
    You might want to turn some of these sub-functions into methods on the class too.
- [ ] (Optional) For this part, if someone else is working on the [paper inputs class](./paper-inputs-class.md), you'll need to talk to them!
  - What was once `paper_vela`, and is now `Residuals.from_timing_residuals`, makes use of three of the "global" input values - `t_0`, `t_g`, and `vela_bin_width`. This isn't ideal and doesn't fit our general `Residuals` class. As such, we need to modify the `from_timing_residuals` function to accept values of `t_0`, `t_g`, and `bin_width` as inputs:

    ```python
    class Residuals:
      ...

      @classmethod
      def from_timing_residuals(
        cls,
        t_max: float,
        *,
        bin_width: float | None = None,
        source: Path | str | None = None,
        t_0: float | None = None,
        t_g: float | None = None,
        ) -> "Residuals":
        ...
    ```

    If values are provided, then the `Residuals` class can use them as normal. If they aren't provided, the `Residuals` class needs to fetch the _default paper input values_ for these variables, and use those instead.
  - If this is proving tricky, consider instead making `from_timing_residuals` have the following syntax:

    ```python
    class Residuals:
      ...

      @classmethod
      def from_timing_residuals(
        cls,
        t_max: float,
        *,
        bin_width: float,
        t_0: float,
        t_g: float,
        source: Path | str | None = None,
        ) -> "Residuals":
        ...
    ```

    that is, with the new arguments being "required". Then, write a separate `classmethod` that handles loading the Vela-dataset:

    ```python
    class Residuals:
      ...

      @classmethod
      def load_vela_data(
        cls,
        t_max: float,
      ) -> "Residuals":
        source = # Fetch the residuals file from the internet.

        t_0, t_g, bin_width = # Default Vela values here (can you fetch them from the PaperInputs class now?)

        return cls.from_timing_residuals(t_max, bin_width = bin_width, source = source, t_0 = t_0, t_g = t_g)
    ```

There are more things we can go with the `Residuals` class:

- You might want to re-work the constructor method; right now we just pass in the attribute values we want to set, but we could just pass in the (array containing the) residuals and the binning information, then compute all the attribute values inside the `__init__` method.
  This might require us moving away from using a `dataclass` altogether.
- You may want to do another pass over the attribute names, and potentially add some useful properties to the class (for example, the number of bins, etc).
