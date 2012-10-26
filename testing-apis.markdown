# Testing API's

{ lang: php }
    namespace Grumpy;

    class GimmebarWrapper
    {
        protected $_apiUrl;

        /**
         * @param string $apiUrl
         */
        public funciton __construct($apiUrl)
        {
            $this->_apiUrl = $apiUrl;
        }

        /**
         * Get the count of public assets if you know the username
         *
         * @param string $username
         * @return integer | false
         */
        public function getPublicAssetCountByUser($username)
        {
            $info = $this->grabPublicAssetsByUser($username);
           
            return $info['total_records'] ?: false; 
        }

        /**
         * Grab public assets for a known user
         *
         * @param string $username
         */
        public function grabPublicAssetsByUser($username)
        {
            $response = file_get_contents(
                $this->_apiUrl . "/public/assets/{$username}"
            );

            return json_decode($response);
        }
    }
}

If there is one question I get over and over again from people seeking
testing advice, it's "how can I test API calls". The only question
that I get asked more is "why are you so grumpy all the time?"

Testing API calls is really no different than testing any other
kind of code: you have a expected output, you execute some code
that calls the API, you verify your test returns the values
that you are expecting.

## Code Construction Is Important
I know this book is supposed to be about using PHPUnit, not about what your
code is supposed to look like. Nonetheless I still think it's important to
understand that the key to really being able to test APIs is by wrapping
code around how you access it.

By this I mean that if you have a library that someone (maybe even you) wrote
to speak to an API, you really should be creating a wrapper around that 
access. Why? For testing purposes, of course!

By using a wrapper that accepts the object(s) that actually speak to the
API, you can create mocks of them when the time comes to do your tests.

Here's an example of what I mean:

{ lang: php}
    class FooApi
    {
        protected $_baseUrl;

        public function __construct()
        {
            $this->_baseUrl = 'http://exaample.com/api';
        }

        public function authorize($username, $password)
        {
            // code that makes an authentication call goes here
        }

        public function call($restVerb, $uri, $data = array())
        {
            /**
             * Code that builds a REST-ful connection to the API
             * would go here
             */
        }
    }

I've seen worse code, don't laugh.

It's also important to understand that you often have no control over the
quality or testability of the code you have to use to speak to a specific
API. I wouldn't panic if the library you are using isn't a work of art.
I save my grumpiness for poorly documenated API libraries.

Now, if we were going to create a wrapper around this, a potential
implementation might look like this:

{ lang:php }
    class FooApiWrapper
    {
        protected $_fooApi;

        public function __construct($_fooApi)
        {
            
        }

        public function authorize($username, $password)
        {
            // code that makes an authentication call goes here
        }

        public function call($restVerb, $uri, $data = array())
        {
            /**
             * Code that builds a REST-ful connection to the API
             * would go here
             */
        }
    }


## Testing Talking To The API Itself
The most common test that people are looking to do is one where you get
data from an API and then you want to transform that data somehow.
Given our example code, it might look something like this:

{ lang:php }
    /**
     * @test
     */
    public function countPublicAssetsByKnownUser()
    {
        $apiUrl = 'https://gimmebar.com/api/v1';
        $gimmebarWrapper = new \Grumpy\GimmebarWrapper($apiUrl);
        $expectedResultCount = 86;
        $publicAssetCount = $gimmebarWrapper->getPublicAssetCountByUser('grumpycanuck');
        $this->assertEquals(
            $expectedResultCount,
            $publicAssetCount,
            'Did not get expected public asset count'
        );
    }

This test is a brittle one because it relies on the API being available at the
exact time we run the test. If you are testing code that needs to gracefully
handle things when the API is not available, this approach to testing
is totally acceptable. 

