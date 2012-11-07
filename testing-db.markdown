# Testing Databases 

{ lang:php }
    namespace Grumpy;

    class Roster
    {
        protected $_db;

        /**
         * @param PDO $db
         */
        public function __construct($db)
        {
            $this->_db = $db;
        }

        /**
         * Method that access the database and returns a summary of the
         * roster for a known team with each player in the format:
         *
         * <shortened team name> <player last name>
         *
         * @param string $nickname
         * @return array
         */
        public function getByTeamNickname($nickname)
        {
            $sql = "
                SELECT tig_name 
                FROM rosters
                WHERE ibl_team = ?";
            $sth = $this->_db->prepare($sql);
            $sth->execute(array($nickname));
            $rows = $sth->fetchAll();
            
            if (!$rows) {
                return array();
            }
            
            $rosterContents = array();

            foreach ($rows as $row) {
                array_push($rosterContents, $row['tig_name']);
            }

            return $rosterContents;
        }
    }  

## Functional Tests vs. Unit Tests

Most web applications are thin wrappers around a database, despite our
attempts to make them sound a lot more complicated than that. If we
have code that speaks to a database, we need to be testing it.

Before we go any further, I want to make a distinction about types of
tests. If you want to be strict about how you are defining your tests, then
if you are writing unit tests, you should never be speaking to the database.

Why? Unit test suites are meant to be testing code, not the ability of
a database server to return results. They also need to run quickly. If your 
test suite takes a long time to run, nobody is going to bother doing it. Who 
wants to wait 30 minutes for your entire test suite to run? I sure don't.
 
If you are testing code that does complex database queries, guess what?
Your test will be waiting every single time you run it for that query to
finish. Again, all those little delays in the execution time for your test
suite add up to people becoming more and more reluctant to run the entire
test suite. This leads to bugs crossing "units" being discovered only when 
someone runs the whole test suite. That's not good enough. We can do better.

I advocate writing as few tests as possible that speak directly to the 
database. If you have some business logic for your application that exists
only in an SQL query, then you probably will have to write a few tests
that speak to the database directly.

After all, I am on interested in testing to see if I can connect to my
database properly. That sort of thing should be written into your application
way before any business logic code gets run. Like in the bootstrap or the
front controller of your framework-based code base.

## Sandboxes
If you are going to write tests that connect to a database, then make sure
that you create a sandbox that the database will live in. When I say
sandbox, I am referring to creating an environment where you can delete
and recreate the database easily. Even better if you can automate doing it.

So make sure that your application supports the ability to decide what
database it will talk to. Set it in the bootstrap, or in your globally-available
configuration object, or in the constructor of the god object every other object
in your application extends itself from. I don't care, just make sure
that you can tell your application what database to talk to.

Not to beat a dead horse, but code that you can inject your dependencies
into makes testing database-driven code a lot easier.

A sample test that talks directly to the database:

{: lang="php" }
    class RosterDBTest extends PHPUnit_Framework_TestCase
    {
        protected $_db;

        public function setUp()
        {
            $dsn = "pgsql:host=127.0.0.1;dbname=ibl_stats_test;user=stats;password=st@ts=Fun";
            $this->_db = new PDO($dsn);
        }

        /**
         * @test
         */
        public returnsRosterSummaryForKnownRoster()
        {
            $roster = new \Grumpy\Roster($this->_db);
            $expectedRoster = array('AAA Foo', 'BBB Bar', 'ZZZ Zazz');
            $testRoster = $roster->getByTeamNickname('TEST');
            $this->assertEquals(
                $expectedRoster,
                $testRoster,
                "Did not get expected roster when passing in known team nickname"
            )
        }
    }

The downside to writing tests that speak directly to a database is that you
end up needing to constantly maintain a testing database. If you have a 
testing environment where multiple developers share the same database, you
run a real risk of over-writing test data or even ending up with data sets
that bear no resemblance to data that actually exists in production.

Like with any kind of testing, you are looking to compare expected results
to the output of your code, given a known set of inputs. The only way to
really achieve this is to either have your testing process dump the existing
database and recreate it from or use database fixtures.

In the PHPUnit world, I feel there is only one database-fixture-handling tool
worth considering.
 
## DBUnit
As I've said before, I am not a big fan of using database fixtures, instead
preferring to write my code in such a way that I instead create objects
to represent the data. Easier to mock, easier to test. But you are not 
necessarily me. If you want to use a database in your tests, I recommend the 
use of [DBUnit](https://github.com/sebastianbergmann/dbunit).

Check the web site for installation details. At the time of writing the
recommended method was via PEAR.

There are times when you do need to talk to a database as part of a test,
usually to verify that if you are saving some information to the database
that you it is still there. Let's look at a way we can create a test that
involves speaking to the database.

### Setting things up for DBUnit
Instead of extending from the usual PHPUnit test case object, we need to use
a different one, and implement two required methods.

Using our sample app, here's one way to do it.

{: lang="php" }
    class RosterDBTest extends PHPUnit_Extensions_Database_Testcase
    {
        protected $_db;

        public function getConnection()
        {
            $dsn = "pgsql:host=127.0.0.1;dbname=ibl_stats;user=stats;password=st@ts=Fun";
            $pdo = new PDO($dsn);

            return $this->createDefaultDBConnection($pdo, $dsn);
        }

        public funciton getDataSet()
        {
            // Load your dataset here
        }

        // Existing tests go below
    }

These two methods we've implemented make sure that any calls to a database
being accessed via PDO will be intercepted by DBUnit.

In case you are wondering, if you omit the getDataSet() method DBUnit will
not truncate your data in the database and replace it the data in your
fixture file. 

Also, when creating your connection in the getConnection() method, make sure
to use the same database credentials that your application is expecting.
Otherwise DBUnit won't intercept calls to the database.

### Using XML datasets
DBUnit supports several types of XML data fixtures. For my tests that
do use them, I like to use "flat XML datasets".

Here's an example:

{: lang="xml" }
   <?xml version="1.0" ?>
    <dataset>
            <rosters id="1" tig_name="FOO Bat" ibl_team="MAD" comments="Test record" status="0" item_type="2" />
            <rosters id="2" tig_name="TOR Bautista" ibl_team="MAD" comments="Joey bats!" status="1" item_type="1" />
            <rosters id="3" tig_name="MAD#1" ibl_team="MAD" status="0" comments="Draft Pick" item_type="0" />
            <rosters id="4" tig_name="TOR Hartjes" ibl_team="MAD" comments="Test writer" status="1" item_type="1" />
    </dataset> 

Then you load it like this:

{: lang="php" }
    public funciton getDataSet()
    {
        return $this->createFlatXMLDataset(
            dirname(__FILE__) . '/fixtures/roster-seed.xml');
    }

If you prefer to be more of a purist, you could create a structured XML dataset.
For our sample dataset, it would look like this:

{: lang="xml"}
    <?xml version="1.0" ?>
    <dataset>
        <table name="rosters">
            <column>id</column>
            <column>tig_name</column>
            <column>ibl_team</column>
            <column>comments</column>
            <column>status</column>
            <column>item_type</column>
            <row>
                <value>1</value>
                <value>FOO Bat</value>
                <value>MAD</value>
                <value>Test record</value>
                <value>0</value>
                <value>2</value>
            </row>
            <row>
                <value>2</value>
                <value>TOR Bautista</value>
                <value>MAD</value>
                <value>Joey bats!</value>
                <value>1</value>
                <value>1</value>
            </row>
            <row>
                <value>3</value>
                <value>MAD#1</value>
                <value>MAD</value>
                <value>Draft pick</value>
                <value>0</value>
                <value>0</value>
            </row>
            <row>
                <value>4</value>
                <value>TOR Hartjes</value>
                <value>MAD</value>
                <value>Test writer</value>
                <value>1</value>
                <value>1</value>
            </row>
        </table>
    </dataset> 

Loading that dataset is very similar:

{: lang="php" }
    public funciton getDataSet()
    {
        return $this->createXMLDataset(
            dirname(__FILE__) . '/fixtures/roster-seed.xml');
    }

One of the drawbacks to using an XML dataset is that if you do have null
values in your data, you have to put a placeholder in your dataset and
then replace it with the desired null value.

Let's say your have a dataset like this:

{: lang="xml" }
   <?xml version="1.0" ?>
    <dataset>
            <rosters id="1" tig_name="FOO Bat" ibl_team="MAD" comments="Test record" status="0" item_type="2" />
            <rosters id="2" tig_name="TOR Bautista" ibl_team="MAD" comments="Joey bats!" status="1" item_type="1" />
            <rosters id="3" tig_name="MAD#1" ibl_team="MAD" status="0" comments="###NULL###" item_type="0" />
            <rosters id="4" tig_name="TOR Hartjes" ibl_team="MAD" comments="Test writer" status="1" item_type="1" />
    </dataset> 
  
{: lang="php" }
    public funciton getDataSet()
    {
        $ds = $this->createFlatXmlDataSet(dirname(__FILE__) 
                . '/fixtures/roster-seed.xml');
        $rds = new PHPUnit_Extensions_Database_DataSet_ReplacementDataSet($ds);
        $rds->addFullReplacement('###NULL###', null);
        
        return $rds;
    }

There are also ways to merge in two different datasets, but we're getting
to the point where I would just be cut-and-pasting the section of the
PHPUnit manual into the book. Not really what I had in mind.

### Using YAML datasets
Don't like XML? You can always do up a data set in YAML

{: lang="yaml" }
    rosters:
      -
        id: 1
        tig_name: "FOO Bat"
        ibl_team: "MAD"
        comments: "Test Record"
        status: 0
        item_type: 2
      -
        id: 2 
        tig_name: "TOR Bautista"
        ibl_team: "MAD"
        comments: "Joey bats!"
        status: 1 
        item_type: 1 
      -
        id: 3 
        tig_name: "MAD#1"
        ibl_team: "MAD"
        comments: 
        status: 0
        item_type: 0 
      -
        id: 4 
        tig_name: "TOR Hartjes"
        ibl_team: "MAD"
        comments: "Test Writer"
        status: 1 
        item_type: 1 

Then, to load that data set:

{ lang:php }
    public funciton getDataSet()
    {
        return new PHPUnit_Extensions_Database_DataSet_YamlDataSet(
            dirname(__FILE__) . '/fixtures/roster-seed.yml');
    }

### Using CSV datasets
Sure, it's possible:

{ lang="csv" }
    id;tig_name;ibl_team;comments;status;item_type
    1;"FOO Bat";"MAD";"Test Record";0;2
    2;"TOR Bautista";"MAD";"Joey bats!";1,1
    3;"MAD#1";"MAD";null;0;0
    4;"TOR Hartjes";"MAD";"Test Writer";1;1

You can load that dataset this way:

{ lang="php" }
    public funciton getDataSet()
    {
        $dataset = new PHPUnit_Extensions_Database_DataSet_CsvDataSet();
        $dataset->addTable(
            'rosters', dirname(__FILE__) . '/fixtures/roster-seed.csv');
        );
    }

### Array-based datasets
Sometimes you just want to hand out data as an array, and not mess with 
any other file format:

{ lang="php" }
    public funciton getDataSet()
    {
        $dataset = array(
            'rosters' => array(
                array('id' => 1, 'tig_name' => 'Foo Bat', 'ibl_team' => 'MAD', 'comments' => 'Test Record', 'status' => 0, 'item_type' => 2),
                array('id' => 2, 'tig_name' => 'TOR Bautista', 'ibl_team' => 'MAD', 'comments' => 'Joey bats!', 'status' => 1, 'item_type' => 1),
                array('id' => 3, 'tig_name' => 'MAD#1', 'ibl_team' => 'MAD', 'comments' => 'Draft pick', 'status' => 0, 'item_type' => 0),
                array('id' => 4, 'tig_name' => 'TOR Hartjes', 'ibl_team' => 'MAD', 'comments' => 'Test Writer', 'status' => 1, 'item_type' => 1)
            )
        )

        return Grumpy_DBUnit_ArrayDataSet($dataset)
    }

The only catch is that we have to implement our own dataset code...

{ lang="php" }
    require_once 'PHPUnit/Util/Filter.php';

    require_once 'PHPUnit/Extensions/Database/DataSet/AbstractDataSet.php';
    require_once 'PHPUnit/Extensions/Database/DataSet/DefaultTableIterator.php';
    require_once 'PHPUnit/Extensions/Database/DataSet/DefaultTable.php';
    require_once 'PHPUnit/Extensions/Database/DataSet/DefaultTableMetaData.php';

    PHPUnit_Util_Filter::addFileToFilter(__FILE__, 'PHPUNIT');

    class Grumpy_DbUnit_ArrayDataSet extends PHPUnit_Extensions_Database_DataSet_AbstractDataSet
    {
        protected $tables = array();

        public function __construct(array $data)
        {
            foreach ($data AS $tableName => $rows) {
                $columns = array();
        
                if (isset($rows[0])) {
                    $columns = array_keys($rows[0]);
                }

                $metaData = new PHPUnit_Extensions_Database_DataSet_DefaultTableMetaData($tableName, $columns);
                $table = new PHPUnit_Extensions_Database_DataSet_DefaultTable($metaData);

                foreach ($rows AS $row) {
                    $table->addRow($row);
                }
               
                $this->tables[$tableName] = $table;
            }
        }

        protected function createIterator($reverse = FALSE)
        {
            return new PHPUnit_Extensions_Database_DataSet_DefaultTableIterator($this->tables, $reverse);
        }

        public function getTable($tableName)
        {
            if (!isset($this->tables[$tableName])) {
                throw new InvalidArgumentException("$tableName is not a table in the current database.");
            }

            return $this->tables[$tableName];
        }
    }

In my mind, the only advantage to going through the hassle of creating your
own dataset object is that you end up with a dataset that handles missing
values a lot easier. 

## Our first DBUnit test
How would we write a test for a method that removes a player from a roster?
Inside  Roster  we could add this method:

{: lang="php" }
    /**
     * Method that deletes an item from our roster based on passing in a
     * known primary ID for that record
     *
     * @param integer $itemId
     * @return boolean
     */
    public function deleteItem($itemId)
    {
        $sql = "DELETE FROM teams WHERE id = ?";
        $sth = $this->_db->prepare($sql);
        return $sth->execute(array($itemId));
    }

The following test verifies that, yes, we can delete items from our rosters
table.

{: lang="php" }
    /**
     * @test
     */
    public function testRemoveBatterFromRoster()
    {
        $testRoster = new Roster($this->_db);
        $expectedCount = 3;

        // Database fixture has 4 records in it
        $testRoster->deleteItem(4);
        $rosterItems = $testRoster->getByTeamNickname('MAD');

        $this->assertEquals(
                $expectedCount,
                count($items),
                'Did not delete roster item as expected'
        );
    }

## Mocking Database Connections

So we have tests that are talking to the database directly and have shown
you how to use fixtures to create known data sets. It's time to move up
to the pure unit test level and make use of mock objects so that we don't
have to actually talk to the database any more.


{ lang: php }
    /**
     * @test
     */
    public function returnExpectedRosterUsingMocks()
    {
        $databaseResultSet = array(
            array('tig_name' => 'AAA Foo'),
            array('tig_name' => 'BBB Bar'),
            array('tig_name' => 'ZZZ Zazz));

First, create an example of what the database would give us back.

{ lang: php }
        $statement = $this->getMockBuilder('StdObject')
            ->methods(array('execute', 'fetchAll'))
            ->getMock();
        $statement->expects($this->once())
            ->method('execute')
            ->will($this->returnValue(true));
        $statement->expects($this->once())
            ->method('fetchAll')
            ->will($this->returnValue($databaseResultSet));

Next, create a mock object to represent the object that PDO would give us
back when we run the prepare() method.

My use of StdObject is okay here because for the purposes of this particular
test. It doesn't really matter what type of object the mocked statement is
because we are more interested in what is returned via those two methods.
I've used this trick a few times when dealing with mocked objects that
need to return other objects. 

{ lang: php }
        $db = $this->getMockBuilder('PDO')
            ->disableOriginalConstructor()
            ->methods(array('prepare'))
            ->getMock();
        $db->expects($this->once())
            ->method('prepare')
            ->will($this->returnValue($statement));
           
Finally, create a mocked PDO object and tell it to return our mocked
statement object. Note the use of disableOriginalContructor(). This is needed
because in this particular test I only care about returning the value from
the prepare() method. By not executing the constructor, we don't have to
worry at all about passing in a correctly-formatted DSN like PDO would
normally expect

{ lang: php } 
        $roster = new \Grumpy\Roster($db);
        $expectedRoster = array('AAA Foo', 'BBB Bar', 'ZZZ Zazz');
        $testRoster = $roster->getByTeamNickname('TEST');
        $this->assertEquals(
            $expectedRoster,
            $testRoster,
            "Did not get expected roster when passing in known team nickname"
        )
    }

The rest of the test is the same, except we pass in our mocked PDO object
instead of the one we created in the test's setUp() method.

## Mocking vs. Fixtures

Having now seen the two approaches, when should you use mocks instead of
fixtures? In my experience, mocks are the best way to handle things if
you are manipulating the results you get back from the database.

If your code is simply returning the results straight from the database,
I think that unit testing that code is of little value. You are better off
investing the time writing functional tests that use fixtures or a database
in a known state.

This is also the case if you have chosen to leverage the query language
your database uses to act as your "business logic". Let's say you want
to have a method that returns the count of players on a roster by 
using an aggregation function in your SQL.

What should the test look like?

{ lang: php }
    /**
     * @test
     */
    public function rosterHasExpectedItemCount()
    {
        // Assuming we are using the fixtures from before
        $expectedRosterCount = 4;
        $roster = new \Grumpy\Roster($this->_db);
        $count = $roster->countItemsByTeamNickname('TEST');

        $this->assertEquals(
            $expectedRosterCount,
            $count,
            'countItemsByTeamNickname() did not return expected roster count'
        );
    }

Now to add a method to our Roster class that uses SQL to give us the answer

{ lang: php }
    /**
     * Return a count of items on a roster when you pass in the team
     * nickname
     *
     * @param string $nickname
     * @return integer
     */
    public function countItemsByTeamNickname($nickname)
    {
        $sql = "
            SELECT COUNT(1) AS roster_count
            FROM rosters
            WHERE ibl_team = ?
        ";
        $stmt = $this->_db->prepare($sql);
        $stmt->execute(array($nickname));
        $result = $stmt->fetchAll();

        return $result['roster_count'];
    }
    
What would be the point of mocking this? It really is what I refer to as
a "passthru" function: it's just returning the response of a single action
without doing any manipulation to it. 

This is not worth doing:

{ lang: php }
    /**
     * @test
     */
    public function returnItemCountUsingMockObjects()
    {
        $expectedRosterCount = 4;
        $databaseResultSet = array('roster_count' => $expectedRosterCount);

        $statement = $this->getMockBuilder('StdObject')
            ->methods(array('execute', 'fetchAll'))
            ->getMock();
        $statement->expects($this->once())
            ->method('execute')
            ->will($this->returnValue(true));
        $statement->expects($this->once())
            ->method('fetchAll')
            ->will($this->returnValue($databaseResultSet));
        
        $db = $this->getMockBuilder('PDO')
            ->disableOriginalConstructor()
            ->methods(array('prepare'))
            ->getMock();
        $db->expects($this->once())
            ->method('prepare')
            ->will($this->returnValue($statement));
        
        $roster = new \Grumpy\Roster($db); 
        $count = $roster->countItemsByTeamNickname('TEST');

        $this->assertEquals(
            $expectedRosterCount,
            $count,
            'countItemsByTeamNickname() did not return expected roster count'
        );
    }     

Having tests are good. Having tests for the sake of writing tests just to  use
a specific testing tool is useless.
