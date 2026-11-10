# Writing Negative Tests

A negative test is a test that checks that our code breaks when we expect it to.
These are useful for a couple of reasons; the first being that we might want to prevent our code from being run in certain situations.
For example, if the user provides a negative value for a parameter that we know to be positive, we may want our code to throw an error at that point to alert the user they've done something wrong.
We are helping prevent the user from using our code incorrectly, and potentially saving them some time in figuring out what's gone wrong (and potentially saving them compute time by alerting them to a problem _before_ they start moving memory to GPUs, for example).

The second reason negative testing is useful is because it's very easy to fall into the ethos of "my code ran, therefore it produced something correct"!
However just because your code ran doesn't mean that it's doing something sensible - this can be a particular pain point when you're working with missing or `NaN` data and you're not careful about how you handle it.
In such cases, you often want to check that doing something you think will cause an error _does indeed_ throw an error.
In the `NaN` example above, you might write a negative test to check that Python is throwing an error when your dataset consists entirely of `NaN` values, rather than silently returning some kind of default value.
[See here for a demonstration of why it's important to have these checks, in a real-life example](https://www.thedrive.com/news/37366/why-that-autonomous-race-car-crashed-straight-into-a-wall).

## (Optional) Create some "Normal" Unit Tests

- [ ] Using the same ideas as the [writing parametrised tests](./writing-tests-params.md) task, write a suite of tests for the (function inside) `numerics.every_pair`.
  - Once you have a few test cases, try to parametrise your various test functions in the manner described above.
  - Hint: `every_pair` takes another function as an input - it would be wise to pick function(s) that use their arguments in simple ways, rather than going for something complicated.
  - Hint: if you are struggling to figure out what the inputs to each test case should be, they will be something like: a function `f` that takes 2 (or more) arguments (to be given as the `run` argument to `every_pair`), two arrays corresponding to the values that need to be looped over (`inner`, `outer`), and another input `args` that contains any values that are held constant over the runs.

## Negative Testing

You may have spotted that `every_pair` relies on the user providing `args` correctly, based on the function that is given to the `run` argument.
What do you expect to happen if, for example, `run` doesn't take any extra values, but some `args` are passed anyway?
Can you think of a way to write a test to see if what you expect does indeed happen?
[Hint: `pytest` might be able to assist...](https://docs.pytest.org/en/stable/how-to/assert.html#assertraises).

- [ ] Write a separate test function for your negative test.
  In this test, make sure the `run` function that you pass to `every_pair` takes exactly two arguments, and then attempt to pass a non-empty `args` value to `every_pair` too.
  [Use a `pytest.raises` context to catch the error](https://docs.pytest.org/en/stable/how-to/assert.html#assertraises), and confirm that the code will break if the user attempts to do this.
- [ ] Can you think of any other cases in which we should expect an error?
  Can you implement a negative test that catches these too?
- [ ] (Optional) (hard) Can you parametrise your negative tests?
