# study

Idiomatic table-driven `#[input, output]` test harnesses for Madlib!

[![Madlib Project Badge](https://img.shields.io/badge/madlib-purple?logo=github&logoSize=auto)](//github.com/madlib-lang/madlib) <!-- $MADLIB.projectBadge -->
[![study v0.0.8](https://img.shields.io/badge/v0.0.8-purple?label=version)](//github.com/brekk/study) <!-- $MADLIB.json.version -->

---

Study is a helpful testing standard for table-driven (`#[input, output]`) testing. It allows you to quickly express repeatable test harnesses in a table-style format.

## The idiomatic example

Here's the basic example, for a unary function (one input, one output): 

```madlib
import { aUnaryFunction } from "@/MyProject"
import Study from "study"

Study.report(
  aUnaryFunction,
  "the name of the test for aUnaryFunction",
  [
    #[givenInput, expectedOutput]
  ]
)
```

### The Prelude example, for comparison

To compare, the equivalent using only Prelude / standard library from Madlib:

```madlib
import { aUnaryFunction } from "@/MyProject"
import { test, assertEquals } from "Test"

test("the name of the test for aUnaryFunction", () => {
  return assertEquals(givenInput, expectedOutput)
})
```

In this simple example, Study doesn't offer much benefit.

### The footgun

However, there's a major shortcoming with the default `test`: if you want to add a second assertion, there's an easy to miss footgun:

```madlib
import { test, assertEquals } from "Test"

ab = (a, b) => a / b
test("this test is good", () => assertEquals(ab(3, 4), 3 / 4))
test(
  "this test is bad and won't catch the failing assertion",
  () => {
    assertEquals(ab(3, 4), 42)
    return assertEquals(ab(3, 4), 3 / 4)
  },
)
test(
  "this one is good again, but is blocking and only the first failing assertion will show",
  () => do {
    _ <- assertEquals(ab(3, 4), 42)
    return assertEquals(ab(1, 2), 30)
  },
)
```

👆🏽 `assertEquals` returns a Wish, and we can use `do` notation to bind the assertions to correctly capture them, but they'll still behave fail-first.

### The benefits of Study

Study overcomes this by doing some automatic plumbing for you, so that each row of the table is a standalone `test` under the hood, so no tests block each other.

This gives you some helpful ways of slicing the same tests. Because the values are externalized, it allows you to define multiple things in easy ways.
You can define a repeatable test harness, you can compose harnesses, and more!

## Adding parameters

If you have a function with more than a single input (who doesn't!), you can either apply partially apply parameters, so you're still testing a unary function:

```madlib
import Study from "study"
abc = (a, b, c) => a ++ b ++ c
Study.report(
  abc("hello ", "there "),
  "testing abc with partial application",
  [
    #[ "world", "hello there world" ],
    #[ "madlib", "hello there madlib" ],
  ]
)
```

### Adding parameters with the `caseN*` functions

You can also use Study's `caseN*` functions (`caseN2` up to `caseN7`) to convert a function which has an arity of greater than one into a unary function.

```madlib
import Study from "study"
abc = (a, b, c) => a ++ b ++ c
testABC = Study.report(
  Study.caseN3(abc),
)
testABC(
  "this is the same as before, but where the inputs are being passed in the table rather than being partially applied"
  [
    #[ #["hello ", "there ", "world"], "hello there world" ],
  ]
)
testABC(
  "it's easy to define another instance of the test harness, which can be helpful if you want to change the name of the test, as this does",
  [
    #[ #["specific ", "test ", "case "], "specific test case" ],
  ]
)
```

### We're all just permutations, spanning time

Because this is all just curried functions and partial application, if you had defined a table of expected inputs and outputs for a given function, you could use `Study` to reuse that same harness with a different function, to ensure that there was consistent behavior between interfaces, for instance.

```madlib
ensureSameInterface = Study.report($, $, MY_HARNESS_TEST_TUPLES)
ensureSameInterface(functionOne, "function one matches our test harness")
ensureSameInterface(functionTwo, "function two also matches our test harness")
```

## Testing asynchronous / Wish-returning functions

Study also ships with `asyncReport`, which allows you to test functions which return Wishes.

```madlib
import Study from "study"

multi = map((x) => x * 2)
asyncReport(multi, "asynchronous double", [#[Wish.good(5), 10]])
```

If you want to test against the failure path, you can use `asyncFail`:

```madlib
import Study from "study"
import Wish from "Wish"

multiFail = Wish.mapRej((x) => x * 2)
asyncReport(multiFail, "asynchronous double happens on the failure path", [#[Wish.bad(5), 10]])
```


## Command Line Interface testing

In addition to the main offering, there's tooling for testing command line interfaces available from `Study/Cli`.

```madlib
import Executive from "Study/Cli"

EXPECTED_HELP_TEXT_OUTPUT = "blah blah blah, stakeholders"

presentation = Executive.powerpoint("./build/mytool")
presentation("print help", #[["--help"], EXPECTED_HELP_TEXT_OUTPUT)
```

There's additional options under the hood, see [the source](https://[github.com/brekk/study/blob/main/src/Cli.mad](//github.comstudy/blob/main/github.com/brekk/study/blob/main/src/Cli.mad)) for more.
