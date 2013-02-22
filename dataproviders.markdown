# Data Providers
{: lang="php" }
    <?php

    function dataProvider()
    {
        return array(
            array(1, 'odd'),
            array(2, 'even')
        );
    }

## Why you should use data providers
One of your main goals should always be to write the bare minimum amount of
code in order to solve a particular problem you are facing. This is no different
when it comes to tests, which are really nothing more than code.

One of the earliest lessons I learned when I first started writing what I felt
were comprehensive test suites was to always be on the lookout for duplication
in your tests. Here's an example of a situation where this can happen.

Most programmers are familiar with the [FizzBuzz](http://en.wikipedia.org/wiki/FizzBuzz)
problem, if only because it is commonly presented as a problem to be solved
as part of an interview. In my opinion it is a good problem to present
because it touches on a lot of really elementary basics of programming.

## Look at all those tests
If you didn't know about data providers, what might your tests look like?
 
{: lang="php" }
    <?php
    class FizzBuzzTest extends PHPUnit_Framework_Testcase
    {
        public function setup()
        {
            $this->fb = new FizzBuzz();
        }

        public function testGetFizz()
        {
            $expected = 'Fizz';
            $input = 3;
            $response = $this->fb->check($input);
            $this->assertEquals($expected, $response);
        }

        public function testGetBuzz()
        {
            $expected = 'Buzz';
            $input = 5;
            $response = $this->fb->check($input);
            $this->assertEquals($expected, $response);
        }

        public function testGetFizzBuzz()
        {
            $expected = 'FizzBuzz';
            $input = 15;
            $response = $this->fb->check($input);
            $this->assertEquals($expected, $response);
        }

        function testPassThru()
        {
            $expected = '1';
            $input = 1;
            $response = $this->fb->check($input);
            $this->assertEquals($expected, $response);
        }
    }

I'm sure you can see the pattern:

* multiple input values
* tests that are extremely similar in setup and execution
* same assertion being used over and over

## Creating data providers

A data provider is another method inside your test class that returns an
array of results, with each result set being an array itself. Through
some magic internal work, PHPUnit converts the returned result set into parameters
which your test method signature needs to accept.

{: lang="php" }
    <?php
    public function fizzBuzzProvider()
    {
        return array(
            array(1, '1'),
            array(3, 'Fizz'),
            array(5, 'Buzz'),
            array(15, 'FizzBuzz')
        );
    }

The function name for the provider doesn't matter, but use some common
sense when naming them as you might be stumped when a test fails and
tells you about a data provider called 'ex1ch2', or something else equally meaningless.

To use the data provider, we have to add an annotation to the docblock
preceding our test so that PHPUnit knows to use it. Give it the name of
the data provider method.

{: lang="php" }
    <?php
    /**
     * Test for our FizzBuzz object
     *
     * @dataProvider fizzBuzzProvider
     */
    public function testFizzBuzz($input, $expected)
    {
        $response = $this->fb->check($input);
        $this->assertEquals($expected, $response);
    }

Now we have just one test (less code to maintain) and can add scenarios to our
heart's content via the data provider (even better). We have also learned the
skill of applying some critical analysis to the testing code we are writing
to make sure we are only writing the tests that we actually need.

## More complex examples

Don't feel like you can only have really simple data providers. All you need
to do is return an array of arrays, with each result set matching the
parameters that your testing method is expecting. Here's a more complex example:

{: lang="php" }
    <?php
    public function complexProvider()
    {
        // Read in some data from a CSV file
        $fp = fopen("./fixtures/data.csv");
        $response = array();
        
        while ($data = fgetcsv($fp, 1000, ",")) {
            $response[] = array($data[0], $data[1], $data[2]);
        }

        fclose($fp);

        return $response;
    }

