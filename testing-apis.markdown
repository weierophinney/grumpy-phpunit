# Testing API's
{ : lang="php" }
    <?php
    namespace Grumpy;

    class Gimmebar 
    {
        protected $_apiUrl;

        public function __construct($apiUrl)
        {
            $this->_apiUrl = $apiUrl;
        }

        public function getPublicAssetCountByUser($username)
        {
            $response = $this->grabPublicAssetsByUser($username);
          
            if (isset($response['total_records'])) {
                return $response['total_records'];
            }

            return false; 
        }

        public function grabPublicAssetsByUser($username)
        {
            $response = file_get_contents(
                $this->_apiUrl . "/public/assets/{$username}"
            );

            return json_decode($response);
        }
    }

If there is one question I get over and over again from people seeking
testing advice, it's "how can I test API calls". The only question
that I get asked more is "why are you so grumpy all the time?"

Testing API calls is really no different than testing any other
kind of code: you have a expected output, you execute some code
that calls the API, you verify your test returns the values
that you are expecting.

## Testing Talking To The API Itself
The most common test that people are looking to do is one where you get
data from an API and then you want to transform that data somehow.
Given our example code, it might look something like this:

{ : lang="php" }
    <?php
    public function testCountPublicAssetsByKnownUser()
    {
        $apiUrl = 'https://gimmebar.com/api/v1';
        $gimmebar = new \Grumpy\Gimmebar($apiUrl);
        $expectedResultCount = 86;
        $publicAssetCount = $gimmebar->getPublicAssetCountByUser('grumpycanuck');
        $this->assertEquals(
            $expectedResultCount,
            $publicAssetCount,
            'Did not get expected public asset count'
        );
    }

This test is brittle because it relies on the API being available at the
exact time we run the test. What happens if you can't actually reach this
API from your testing environment? This becomes important if you are 
rate-limited in your access.

Also consider the situation where you access the API through one URL for your
development work but a different one for production. 

## Wrapping Your API Calls 
I know this book is supposed to be about using PHPUnit, not about what your
code is supposed to look like. Nonetheless I still think it's important to
understand that the key to really being able to test APIs is by wrapping
code around how you access it.

By this I mean that if you have a library that someone (maybe even you) wrote
to speak to an API, you really should be creating a wrapper around that 
access. Why? For testing purposes, of course!

This does mean that we will have to refactor the code so that our API class
just returns a raw JSON response, and then we create a wrapper object that
manipulates the API responses.

Yes, this sucks. But testable code is the key.

First, let's refactor our API object:

{: lang="php" }
    <?php
    namespace Grumpy;

    class Gimmebar 
    {
        protected $_apiUrl;

        public function __construct($apiUrl)
        {
            $this->_apiUrl = $apiUrl;
        }

        public function grabPublicAssetsByUser($username)
        {
            $response = file_get_contents(
                $this->_apiUrl . "/public/assets/{$username}"
            );
            
            return json_decode($response);
        }
    }

Next, we create a wrapper object that takes the response from the API object
and then applies some transformations to it. This way, the wrapper doesn't 
actually need to know that it is not dealing with the real thing. It just 
knows it's getting something that it knows how to use.

To isolate code for testing purposes, we should then proceed to create a
mock object representing the real API, and pass that into our wrapper object.

{: lang="php" }
    <?php
    namespace Grumpy;

    class GimmebarWrapper 
    {
        protected $_api;

        public function __construct($api)
        {
            $this->_api = $api;
        }

        public function grabPublicAssetsByUser($username)
        {
            return $this->_api->grabPublicAssetsByUser($username);
        }

        public function grabPublicAssetCountByUser($username)
        {
            $response = $this->_api->grabPublicAssetsByUser($username);

            if (isset($response['total_response'])) {
                return $response['total_response'];
            }

            return false;
        }
    }

Okay, so now that we have a wrapper that accepts our newly refactored
API object, let's write a test to verify that stuff is working the way that
we expect.

{: lang="php" }
    <?php
    public function testCorrectlyFormatResponseFromApi()
    {
        // Sample JSON response from Gimmebar API
        $apiResponse = <<EOT
        {
           "more_records": true,
           "records":[
              "id":"506e217c29ca15dc58000025",
              "asset_type":"page",
              "content":{
                 "full":"http:\/\/gimmebar-assets.s3.amazonaws.com\/506e217a50d77.html",
                 "original":"http:\/\/merrickchristensen.com\/articles\/javascript-dependency-injection.html",
                 "fullscreen":"http:\/\/s3.amazonaws.com\/gimme-grabs-new\/18AF8B43-E8E5-40F9-BA4C-92FC4ADA9DA1",
                 "thumb":"http:\/\/s3.amazonaws.com\/gimme-grabs-new\/CACED21F-2D18-453F-AEBF-E4B79802E88C"
              },
              "date":1349394812,
              "media_hash":"c1cdc3e3fb3da302162ecb990a0dfa9216217",
              "private":false,
              "source":"http:\/\/merrickchristensen.com\/articles\/javascript-dependency-injection.html",
              "description":"",
              "tags":[
              ],
              "size":16217,
              "mime_type":"text\/html",
              "username":"grumpycanuck",
              "user_id":"b7703f2a12d4c7e1cc2f6999e593e3d0",
              "title":"Merrick Christensen - JavaScript Dependency Injection",
              "short_url":"http:\/\/gim.ie\/3PxM"
             ],
           "total_records": 1,
           "limit": 10,
           "skip": 0 
        },
        EOT; 
        
        $api = $this->getMockBuilder('\Grumpy\GimmebarApi')
            ->setMethods(array('grabPublicAssetsByUser'))
            ->getMock();
        $api->expects($this->once())
            ->method('grabPublicAssetsByUser')
            ->will($this->returnValue($apiResponse);

        $apiWrapper = new \Grumpy\GimmebarWrapper($api);
        $testResponse = $apiWrapper->grabPublicAssetsByUser('chartjes');
        $this->assertTrue(is_array($testResponse));
    } 

In this test case we are making sure that the Gimmebar wrapper is correctly
handling a typical response that we from Gimmebar itself. Here's another 
example of a test using a mock object

{: lang="php" }
    <?php
    public function testCorrectlyCountPublicAssets()
    {
        // Sample JSON response from Gimmebar API
        $apiResponse = "{'total_records': 10}";
        $api = $this->getMockBuilder('\Grumpy\GimmebarApi')
            ->setMethods(array('grabPublicAssetsByUser'))
            ->getMock();
        $api->expects($this->once())
            ->method('grabPublicAssetsByUser')
            ->will($this->returnValue($apiResponse);

        $apiWrapper = new \Grumpy\GimmebarWrapper($api);
        $expectedCount = 10;
        $testCount = $apiWrapper->getPublicAssetCountByUser('test');
        
        $this->assertEquals(
            $expectedCount,
            $testCount,
            "Did not correctly count the number of public assets"
        );
 
Just like any other test, we're still following the same logic: we create
a scenario, mock out resources that are required for that scenario, and 
then test our code to make sure that based on a known set of inputs
that we are getting an expected output.

Pay attention to the fact that in order to test this particular bit of
functionality, we don't even need a full response. Just a fake response
containing the data only required is all it takes.
