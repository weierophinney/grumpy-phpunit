# PHPUnit For Grumpy Developers
PHPUnit can look intimidating to just get the skeleton of a test created due
to the immense length of it's documentation and large number of configurable
options.

Don't be scared, I am here to help you! You can start with just a few of the
basics and then move on to more complicated setups that include options for
skipping certain types of tests or changing default settings.

To be honest, the defaults will cover 99.999% of your testing needs.

## Installing And Configuring
Installing PHPUnit and it's associated (sometimes optional) dependencies has
gotten easier and easier with each passing day. As I write this book in the
Canadian winter of 2012-2013, I have two preferred ways of installing PHPUnit that I
would consider.

### Installing via Composer
[Composer](http://getcomposer.org/) is a command-line tool for tracking and
installing dependencies for your application. It is my opinion that Composer
is a tool that is providing a real transformation in the way PHP developers
are building their PHP applications by making it easier to install dependencies
and other people's libraries. With all the major frameworks supporting it, 
there is no reason to not use it if you are using PHP 5.3 or greater.

To install it using Composer, once you've installed Composer itself then
you create a JSON file that tells Composer where you want it installed.
When I use Composer, I tend to install all my dependencies locally but not
add them to my project via version control, instead relying on my build
tools to pull in the dependencies during deployment time.

I also highly recommend NOT including PHPUnit in your Composer configuration
file when you are loading it in production. You can specify a name for
the configuration file when you run Composer, so having a development version
is handy. 

A sample composer.json file would look like this:

{: lang="json" }
    {
        "require-dev": {
            "phpunit/phpunit": "3.7.*"
        }
    }

That's all you need to get the latest version of PHPUnit as of this writing.
Check the PHPUnit documentation for further details on installing PHPUnit
system wide if that's your preferred method.

Composer will also try to pull in any required dependencies, but if for
some reason they won't work you can just add them to composer.json.

### Installing Via PEAR
[Pear](http://pear.php.net) used to be my preferred method of installation
before they made it available via Composer. In this case, installing things
can be as simple as:

{: lang }
    path/to/pear config-set auto_discover 1
    path/to/pear install pear.phpunit.de/PHPUnit 

By default PEAR will try to pull in additional dependencies for PHPUnit, 
but you can install the additional missing components via PEAR as well.

### Which One Should I Use
That depends on what additional dependencies you happen to be including. I 
think that you should stick with one method, as mixing Composer and PEAR
might cause you to make mistakes and forget a package that you are likely
to need. Composer installs dependencies locally by default while PEAR
does global installs.

## Minimum Viable Test Class 
{: lang="php" }
    class GrumpyTest extends PHPUnit_Framework_TestCase
    {
        public function testMinimumViableTest()
        {
            $this->assertTrue(false);
        }
    }

That is what I would call a Minimum Viable Test class. All test classes
need to extend off of the base PHPUnit_Framework_TestCase class, although
it is common for people to create their *own* base class that extends from
this one, and then all their test cases extend from it. Sounds like we are
getting to [Inception](http://en.wikipedia.org/wiki/Inception) levels of
class construction. 



