## My most used mocking techniques in jest


Mocking in jest (or practically any other testing framework for that matter)
can sometimes feel a bit overwhelming. First of all, the terminology is
confusing and often varies a) from framework to framework and b) from textbook
to textbook / educator to educator. Here are some of the basic mocking use
cases I have found myself googling for again and again.

## Some terminology

To help with the terminology, I am referencing Vladimir Khorikov's book [Unit
Testing: Principles, Practices, and
Patterns](https://www.manning.com/books/unit-testing) (Manning 2020). Khorikov
(2020: 93) notes that _test double_ is actually a good generic term: it refers
to any kind of fake dependency used in tests. Generally, at least the js
community seems to be using _mock_ in the role of a catch-all term and _double_
is a rare sight. This is a bit problematic at least in the light of Khorikovs
definition, since he also makes the following distinction:

- _Mocks_ are used for emulating outcoming interactions
- _Stubs_ are used for emulating incoming interactions

This distinction would mean that mocks are something that don't need to return
any values (`jest.fn()`) and that are used for making sure your app correctly interacts with
it's dependencies (i.e. sends the right parameters). Khorikov's basic example
for this is mocking sending an email to an smtp server. Notably, _spies_ are a
subcategory of mocks.

Stubs, on the other hand, would in this distinction be something that return
values used in your application. Using stubs is a way to test how your
application reacts to different kinds of inputs: a typical case would be mocking a
database's response to a query for a list of items.

Unlike e.g. mocha, jest doesn't use the term _stub_ at all. _Mock_ and _spy_
are part of jest's vocabulary, but different use cases require different usage
patterns of `jest.mock` and `jest.spyOn`.

## Basic use cases

Here's my list of basic use cases for test doubles using jest. Often there are
multiple ways of doing the same thing, but here's what I tend to use often.
What usually causes problems, is the fact that jest is still quite poor at
handling es modules. Test setups with ts-jest tend, in my experience, to be
simpler than the ones involving babel, but this may vary. What also matters is
whether or not you're mocking / stubbing an import from inside your own codebase or from
an external package.

### Use case 1: Stubbing a named export from inside the project

This is a case for a stub rather than a mock: you want to replace a function exported as a named export with some test values.  The basic trick is to import the whole module with an alias (`services` in this case) and then use `spyOn` for that method. If a more generic stub used in every test case would be needed, you could also run `jest.mock("services", ....)` and replace `...`with a factory function for each named export.

```typescript
import * as services from "./services";

//......

describe("...", () => {
  it("...", () => {
    jest.spyOn(services, "getListOfSomething").mockResolvedValue(sampleList);
  });
});
```

### Use case 2: Mocking a default export from an external library

This is the straight-forward mocking use case (just recording call parameters, not worrying about the output given to our app) that rarely causes trouble.

```typescript
import axios from "axios"
jest.mock("axios");

describe("...", () => {
  it("...", () => {
    expect(axios.get).toHaveBeenCalledWith(...)
  });
});


```

Note: also works for mocking a default export in your own project

### Use case 3: Mocking a class with a constructor

Sometimes you need to mock a class instead of a single function: `jest.fn().mockImplementation` is your friend here. This is also a mock rather than a stub: we just need referring to window.ResizeObserver not to cause any errors in our test run, which is why we're using jest.fn for the actual methods.


```typescript
window.ResizeObserver = jest.fn().mockImplementation(() => ({
  observe: jest.fn(),
  unobserver: jest.fn(),
  disconnect: jest.fn(),
}));
```

### Use case 4: stub and specify a different output per test

I find it often to be the case that you can't just stub something once
and then use that output. Many times the very reason for stubbing is testing
how your app reacts to different kinds of inputs. This is the pattern I tend to
use:

1. Import the function you're stubbing
2. Add a generic mock with `jest.fn`
3. Inside the test cases, specify a different return value (note the casting
   needed for typescript; btw ts-node has a shortcut for this)

```typescript
import { someFunc } from "some-lib";
jest.mock("some-lib", () => ({
  ...jest.requireActual("some-lib"),
  someFunc: jest.fn(),
}));

//...

it("does x if y", () => {
  (someFunc as jest.Mock).mockReturnValue(returnValue1);
  const output = someFunc(someParam);
  expect(output).toEqual(1);
});

it("does a if b", () => {
  (someFunc as jest.Mock).mockReturnValue(returnValue2);
  const output = someFunc(someParam);
  expect(output).toEqual(2);
});
```

## More Caveats

Sometimes mocking fails because the order of when the mocking is done matters.
A common case is when mocking e.g. a backend middleware and testing REST
endpoints using supertest or something similar. In that case, you need to make
sure that mocking the middleware happens before importing the express/koa app
and launching it in your test setup function.
