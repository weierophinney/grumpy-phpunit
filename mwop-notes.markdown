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
------------------------------

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

command line switches
^^^^^^^^^^^^^^^^^^^^^

Introduce this section:

    Command line switches may be specified in the configuration file as
    attributes of the root `phpunit` element. The following provides
    configuration for the `--backupGlobals` and `--processIsolation` switches,
    respectively:

Also, link to Appendix C of the PHPUnit documentation; don't just mention it.

process isolation
^^^^^^^^^^^^^^^^^

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
