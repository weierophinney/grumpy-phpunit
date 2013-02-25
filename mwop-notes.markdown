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
