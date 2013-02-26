Notes from mwop
===============

General
-------

I've been doing a commit per change in this branch; see the commit messages to
understand the change I made.

Formatting
^^^^^^^^^^

I'd recommend marking class names, method names, etc. within text as code, using
the backtick markers (e.g., "`PHPUnit_Framework_TestCase`"). 

phpunit-for-grumpy-developers.markdown
--------------------------------------

installing via composer
^^^^^^^^^^^^^^^^^^^^^^^

A note: if you use "require-dev", you're actually not forcing people to install
PHPUnit in order to use your project. Any requirements that are marked
"require-dev" will only be installed if you pass the "--dev" flag when
installing or updating. This is great when simply consuming a library/project,
as you can omit development-only dependencies. If they _do_ want to run tests,
however, they can be certain they're getting a version of PHPUnit that works for
the tests, and that it's installed.

minimal viable test class
^^^^^^^^^^^^^^^^^^^^^^^^^

*Why* do you feel creating a custom base class leads to "Inception levels of
class construction"? I've had very real use cases for doing this in the past,
and it would be very useful to hear your opinions on why it's a bad practice (or
why not).

making your tests tell you what's failed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

I'm not sure why you're talking about testdox here, to be honest. *When a failure
or error occurs, PHPUnit already reports the method name, as well as the line
number of the test class that raised the error or failure.* Testdox simply gives
you a "readable" way of seeing those method names, as well as indicating inline
the status of that particular test.

Instead of talking about testdox here, I'd recommend using the `--verbose` flag
with PHPUnit; its stated purpose is to give information on what tests were
marked incomplete or were skipped -- which would give the information necessary
to go back and investigate if need be.

configuring run time options
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

I'd suggest marking the options also as code (in addition to being bold); makes
it more clear that these are something you'll type.

*logging options*

It's a bit wierd to talk about logging for Jenkins without indicating what CI
is, why you'd have it setup, etc. It might be nice to have an appendix about CI
options, why you'd use them, etc., and link to that from this section.

Also, this might be a better place to talk about testdox, as it's really more of
a logging format anyways.

*code coverage options*

An additional point to make about code coverage reports is that you can use them
to identify areas of your code that are not yet tested. This can be useful
during code review, as you can get a feel of what edge cases may still exist, or
when adding features/fixing bugs, where somebody skimped in the past, and thus
unknown expectations may exist.

test environment configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

"It will quickly get tedious" -- _what_ will quickly get tedious? Rephrase. I'd
suggest: "You will find manually adding command-line switches quickly becomes
tedious."

Additionally, provide a transition before discussing `phpunit.xml`. I'd rephrase
the first two paragraphs to read:

    You will find manually adding command-line switches when running your tests
    quickly becomes tedious. Fortunately, PHPUnit allows you to use a
    configuration file for specifying the default switches, among other
    settings. By default, PHPUnit will look for a file named either
    `phpunit.xml` or `phpunit.xml.dist` in the directory in which you run it,
    and use the values it contains to alter its own behavior. 

    You may need different configuration for different kinds of tests -- e.g.,
    unit tests vs. integration tests. PHPUnit allows you to indicate a specific
    configuration file using the `--configuration` switch, with an argument
    indicating the path of the configuration file.

The above introduces the concept more thoroughly, which should answer more
questions out of the gate.

*command line switches*

Introduce this section:

    Command line switches may be specified in the configuration file as
    attributes of the root `phpunit` element. The following provides
    configuration for the `--backupGlobals` and `--processIsolation` switches,
    respectively:

Also, link to Appendix C of the PHPUnit documentation; don't just mention it.

*process isolation*

3rd paragraph, beginning with "More commonly": I'd rephrase this, as I was
confused -- were you saying you were seeing it more commonly? or that the
practice is "more commonly" seen when doing integration tests? I think it's the
latter, in which case I'd rewrite to something like:

    Process isolation smoothes over these problems, as it allows tests to run in
    their own requests. This is particularly interesting when writing
    integration tests, where you will manipulate real objects, not test doubles,
    and state therefor becomes an important consideration.

4th paragraph, beginning "To insist on" -- that's an odd phrasing. I'd use "To
enforce process isolation".

5th and 6th paragraphs: clarification: `@runInSeparateProcess` at the
_class_-level docblock will force test isolation for all tests in that class.
However, if you put it as a _method_-level annotation, it only affects that
specific test method. (This is different than your claim that its placement
affects "all following tests" -- it depends on the context of the docblock.)

I think you also need to better explain the "global state" hack and why it's
needed. Basically, if you _are_ doing test isolation, you lose global state
between test runs, requiring the hack.

7th paragraph -- you're now confusing readers -- in the example, you set
`preserveGlobalState` to `true`, but now you advocate `false` -- explain.

8th and last paragraphs: give an explicit example of this. While I understand
what you're getting at, I don't think you can assume that your readers are as
familiar with `phpunit.xml` and its different settings. Demonstrate _both_ files
-- both the one for tests that should run in isolation, and the one for those
that should not -- so that readers can have a full example.

organizing your tests
^^^^^^^^^^^^^^^^^^^^^

*file system*

Explain _why_ you organize your tests this way. Maybe even detail other ways to
organize tests (I've seen many advocate having your tests _with_ the code being
tested, to keep it all in the same place; PHPUnit will still find only those
classes that are test cases in that situation.)

*xml configuration*

You should:

- indicate that *you don't need to define test suites*; the default behavior of
  PHPUnit is to run all test cases found.
- indicate what a "test suite" is in terms of PHPUnit execution, and how you
  would invoke a specific named test suite.
- indicate that you can have multiple test suites defined (which is implied
  anyways, but will also lead into the next section better).

*multiple test suites*

Demonstrate running _both_  test suites, and potentially even show some dummy
output; this will make it more clear that different sets of tests are being run.

Also, in the example from the previous section, you show building a test suite
with multiple individual files -- but now show it using directories. I think a
more comprehensive explanation of what configuration is allowed in a test suite,
and what the implications of each configuration style is, would be really nice
here.

test-doubles.markdown
---------------------

I'd have more of an introduction here. Detail:

- Basics of dependency injection
- Why DI is an important aspect of testing (i.e., why it makes testing easier)
- The new "problem" DI creates when testing

This all then leads in to defining test doubles: a way to "fake" the
dependencies that you inject into the subject under test, so that you can focus
on testing the subject, not all of its interactions.

general notes
^^^^^^^^^^^^^

For each of the sections on the individual test double types, I'd recommend the
following structure:

- Introductory text defining the problem that the given test double type
  attempts to solve.
- Attempt to define the test double type in a single, highlighted sentence.
- Provide a piece of code you will use to demonstrate.
- Discuss how you will use the test double strategy to solve it, including
  introducing any new PHPUnit concepts related to that strategy.
  - For example, introduce the `getMockBuilder()->getMock()` chain with dummy
    objects, the `expects()->method()->will()` chain with stubs, etc.
- Show the full test solution.

On skimming the sections, it's not immediately clear what each test double is
solving within the tests - it often looks like the same examples. Differentiate
them more, so the readers learn what each strategy is, when to use it, and what
tools are available to make it happen.

Also: *you did not write a section on test fakes!* Either omit it from the
intro, or write a section on it. :)

dummy objects
^^^^^^^^^^^^^

This section starts out poorly. You start with a code sample, but there's no
explanation of it or how it relates to the section title. Then, in the first
paragraph of text, you talk generically of test doubles, and then mention that
you can "just create a dummy object," without ever defining the term.

2d and 3rd paragraphs: Don't get too complicated right off the bat. Omit the "we
are only testing code that exercises one of those dependencies" bit -- just
focus on the dummy object. We're testing an object that has a required
constructor dependency, but we don't need that dependency to do anything. In the
example, then, call a method that does not utilize that dependency, and do an
assertion on that.

Last paragraph: you talk about not needing to mock out all of `Foo`, and how you
shuld not mock things if they don't need to be mocked -- but this is all done
without having introduced the concept of mocking. Omit it. Simply reiterate what
a dummy object is: a required dependency of the object or method under test, but
which is never invoked in the test itself.

test stubs
^^^^^^^^^^

While paragraph 2 is a nice explanation of what test stubs *are*, I'd rework the
next 4 paragraphs to go through the "what" and "how" with more detail:

- Tell it how many times/when it will be invoked (`expects()`)
- Tell the mock what we want to run (`method()`)
- Tell it what to return when the method is called (`will()`)

I'd also note that once you have an `expects()` method, you're implicitly
declaring an assertion. If that method is never called, or not called with the
arguments specified, the test will fail. This is an important note; I've seen
people in code review clamor about a lack of assertions -- even though a mock is
clearly in play.

If it's possible to create a sidebar or a "note", move the paragraphs on
protected/private methods into that; right now, they simply interrupt the flow.
(And since you have a section on this already, you could just omit these
paragraphs entirely.)

Paragraph beginning with "In my experience" should be the first paragraph of
this section.

Last paragraph is good, but maybe an illustration would be even better.

*expectations during execution*

The last paragraph is incredibly cryptic; and this is coming from somebody who
has used `at()`. Detail it a bit more:

- `at()` expects an integer indicating which sequential invocation of the method
  it will respond to.
- integers begin with 0
- any `will()`, `method()`, and/or `with()` chained here will be associated with
  that specific invocation.

Might be good to also note that if you call `once()`, it will look for the
method to be executed once with precisely the aruments specified.

*returning specific values based on input*

I'd add some intro text in here, something along the lines of, "Often, the
methods you invoke on dependencies accept arguments, and vary the return value
based on what they receive. With PHPUnit, you can tell a mock object what to
expect for arguments, and bind a specific return value for that input. To do
this, you use the `with()` method to detail arguments, and the `returnValue()`
method, inside the `will()` method, to detail return values."

Regarding the last paragraph: IIRC, `expects(once())` doesn't dictate order; you
can have the various `expects(once())` calls in any order, and as long as each
is called, the test will pass. You use `expects(at())` to dictate order.

*testing protected and private methods of objects*
*testing private and protected attributes*

Personal preference: I wouldn't prefix non-public members with an underscore. It
makes refactoring harder later.

I'd also explain the toolchain a bit:

- Use the ReflectionAPI to get a reflection method object
- call `setAccessible(true)` on the reflection method object you're testing
- call `invoke()` on the reflection method object with the arguments you want to
  pass to it

Next, when you want to do assertions against a protected attribute, you can use:

    assertAttribute(Not)?(Count|Empty|GreaterThan|LessThan|Equals|Contains|Same|InstanceOf|InternalType)(Only|OrEqual)?($attribute, $object, $testValue, $message)

These work even on non-public members, which is far simpler than using
readAttribute() and doing an assertion. It also allows you to omit the "testing
private and protected attributes" section -- or you can replace that section
with a description of the methods available, and how you'd use them.

test spies
^^^^^^^^^^

This section needs more information. Detail how the approach is different than
test stubs: you're not testing return values, only that you're called a set
number of times; optionally, that you received specific values in a certain
order (this would be where to introduce `at()`).

Also, expand your definition of `expects()` here -- that it can also accept an
integer indicating a specific number of times it should be executed.

testing traits
^^^^^^^^^^^^^^

Cool section. :) 

I'd maybe detail the problem a bit more.

    Because Traits can be defined once, but used many times, you will not want
    to necessarily test the functionality defined in traits in every object in
    which they are consumed. At the same time, you _do_ want to test the traits
    themselves. PHPUnit 3.6 and newer offers functionality for mocking traits
    via the `getObjectForTrait()` method; this will return an object composing
    the trait, so that you can unit test only the trait itself.

dataproviders.markdown
----------------------

Needs an introduction: what is a data provider? 

I'd define them, and then define the problem they solve, by merging the "why you
should use data providers" and "look at all those tests" sections.

creating data providers
^^^^^^^^^^^^^^^^^^^^^^^

First paragraph: actually, the data provider doesn't need to be a method inside
the test class. It can come from a variety of sources: a file, static class
method, function call, etc. You can then argue why you prefer methods within the
class.

I'd also detail what the structure is, and why -- why each element of the
returned array has to be an array. Related, detail naming the arguments to the
test method itself, so that there is a clear semantic relation between what is
provided, and how the test method consumes it.

Also, some information you didn't cover, which I've found immensely useful: if
you return an associative array, the keys will be used to name the data sets,
which makes debugging soooo much easier:

    return array(
        'one'      => array(1, '1'),
        'fizz'     => array(3, 'Fizz'),
        'buzz'     => array(5, 'Buzz'),
        'fizzbuzz' => array(15, 'FizzBuzz')
    );

more complex examples
^^^^^^^^^^^^^^^^^^^^^

Spot on. :) I've done similarly in tests I've written, calculating values ahead
of time, even creating closures over variables.

creating-test-data.markdown
---------------------------

serialize and store
^^^^^^^^^^^^^^^^^^^

Provide the Widget class declaration, so that readers can see why it can be
serialized (i.e., there are no hidden dependencies, no dependencies on
resources, etc.). This makes the concept cleaner.

testing-apis.markdown
---------------------

general notes
^^^^^^^^^^^^^

As noted elsewhere: personal preference is not to prefix non-public members.

I'd likely also link to other resources, as this is a topic for which there are
many, many approaches. You know, like the PHP QA book. :)

testing talking to the api itself
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

2d paragraph: it's brittle not just because the API needs to be available, but
because you need to be certain the API will return what you expect.
