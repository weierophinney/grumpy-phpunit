# Testing databases 

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
a different onee, and implement two required methods.

Using our sample app, here's one way to do it.

{: lang="php" }
    class RosterDBTest extends PHPUnit_Extensions_Database_Testcase
    {
        public function getConnection()
        {
            $dsn = "pgsql:host=127.0.0.1;dbname=ibl_stats;user=stats;password=st@ts=Fun";
            $pdo = new PDO($dsn);

            return $this->createDefaultDBConnection($pdo. $dsn);
        }

        public funciton getDataSet()
        {
            return $this->createFlatXMLDataset(dirname(__FILE__) . '/fixtures/roster-seed.xml');
        }
    }

Yes, you are going to have to use XML. Deal with it.

Unfortunately, DBUnit only really supports working with PDO connections. I did
end up refactoring code in the sample application to use it. Luckily we don't
need to actually alter our code in order to use the fixtures. 

These two methods we've implemented make sure that any calls to a database
being accessed via PDO will be intercepted by DBUnit.

In case you are wondering, if you omit the getDataSet() method DBUnit will
not truncate your data in the database and replace it the data in your
fixture file. 

Also, when creating your connection in the getConnection() method, make sure
to use the same database credentials that your application is expecting.
Otherwise DBUnit won't intercept calls to the database.

### Creating a data fixture

Next, we need a data fixture. Easiest way is to use an XML-based fixture file.
Why else do you think I picked it?

Here's one.

{: lang="xml" }
   <?xml version="1.0" ?>
    <dataset>
            <teams id="1" tig_name="FOO Bat" ibl_team="MAD" comments="Test record" status="0" item_type="2" />
            <teams id="2" tig_name="TOR Bautista" ibl_team="MAD" comments="Joey bats!" status="1" item_type="1" />
            <teams id="3" tig_name="MAD#1" ibl_team="MAD" status="0" item_type="0" />
            <teams id="4" tig_name="TOR Hartjes" ibl_team="MAD" comments="Test writer" status="1" item_type="1" />
    </dataset> 

So why go to the trouble of doing this? Going with fixtures means that you 
can create datasets that are customized for the code you wish to test
without having to create an actual test database. It also means you can
update your data fixtures as you make changes.

### Our First DBUnit Test
That gives us some records to start. Let's add in a test to grab all the 
players that exist in our database and verify that we've gotten all of
them (in this case, 2 of them).

{: lang="php" }
    public function testGetExpectedBatterCount()
    {
        $db = DB::connect(DSN);
        $testRosterModel = new RosterModel($db);
        $expectedCount = 2;
        $batters = $testRosterModel->getByNicknameAndType('MAD', 1);

        $this->assertEquals(
            $expectedCount,
            count($batters),
            'Did not get expected batter count for roster'
        );
    } 

How would we write a test for a method that removes a player from a roster?
Inside  RosterModel.php we could add this method:

{: lang="php }
    public function deleteItem($itemId)
    {
        $sql = "DELETE FROM teams WHERE id={$itemId}";
        
        return $this->_db->query($sql);
    }

The following test verifies that, yes, we can delete items from our rosters
table.

{: lang="php" }
    public function testRemoveBatterFromRoster()
    {
        $testRosterModel = new RosterModel($this->_db);
        $expectedCount = 1;

        // id 4 from our dataset is a batter
        $testRosterModel->deleteItem(4);
        $items = $testRosterModel->getByNicknameAndType('MAD', 1);

        $this->assertEquals(
                $expectedCount,
                count($items),
                'Did not delete batter item as expected'
        );
    }

