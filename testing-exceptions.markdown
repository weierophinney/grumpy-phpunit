# Testing Exceptions

{ lang: php }
    <?php
    class Foo
    {
        public function __construct($badWordsArray)
        {
            $this->_badWordsLIst = $badWordsArray;
        }
        
        public function censorCheck($word)
        {
            if (in_array($word, $this->_badWordsArray)) {
                throw new FooException("Found a bad word!");
            }

            return true;
        }
    }

If you're into writing what I refer to as "modern PHP", you are definitely
going to want to be using exceptions to trap all your non-fatal errors. 
Code that has exceptions also needs to be tested. Never fear, PHPUnit can
show you the way.

## Marking Exceptions For Testing

PHPUnit relies on the use of annotations to indicate what exceptions it
is expecting to encounter when testing code. Let's create a test for our
code sample above.

{ lang: php }
    <?php
    /**
     * Test that makes sure we are correctly triggering an
     * exception by matching words on our list of bad words
     *
     * @expectedException FooException
     */
    public function testThrowsCorrectException()
    {
        $word = "Alpha";
        $badWords = array("Alpha", "Beta", "Gamma", "Zeta");
        $foo = new Foo($badWords);
        $foo->censorCheck($word);
    }

When this test gets run, it will expect that in the course of executing
your code an exception of the type FooException will be thrown.




