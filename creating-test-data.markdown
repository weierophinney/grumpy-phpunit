# Creating Test Data
If you are testing functionality that needs data to manipulate,
you need to learn how to create and provide realistic data for your
tests.

Note the key word here is "realistic." Even the best tests are
of no use to you if they consume data that is wildly different
from the data your code uses in production.

## Data Source Snapshots
If you are using PHPUnit to do integration tests, or
haven't really written unit tests that use test doubles
to simulate speaking to a data source, then investing
time in scripts that can take snapshots of your data
will pay off.

### Shell scripts
One of the easiest ways I know to get large amounts
of data out of a data source easily is to use whatever
CLI utilities are provided with it. The classic
example is using *mysql_dump* to grab tables and
then the MySQL CLI tool to import the data into
the database associated with your tests.

Any tool that can automate the process of
getting the raw data from one location to another
will be of use to you.

### Serialize And Store
I have a personal preference for architectures where I speak to data
sources and create data objects from the results. One benefit of
this is that, during testing, I can create collections of objects
by writing some PHP code and serializing the results to the
file system for later retrieval.

Here's an example:

{ lang: php }
    <?php
    // $dbh is a PDO DB object
    $sql = "SELECT * FROM widgets WHERE type = 'standard'";
    $stmnt = $dbh->prepare($sql);
    $stmnt->execute();
    $results = $stmnt->fetchAll(PDO::FETCH_ASSOC);
    $collection = array();

    foreach ($results as $result) {
        $widget = new Widget();
        $widget->sku = $result['sku'];
        $widget->description = $result['description'];
        $widget->color = $result['color'];
        $widget->price = $result['price'];
        $collection[] = $widget;
    }

    $data = serialize($collection);
    file_put_contents(
        './tests/fixtures/widget-collection.txt',
        $data
    );

When you're ready to use it in your test:

{ lang: php }
    <?php
    $data = file_get_contents('./test/fixtures/widget-collection.txt');
    $collection = unserialize($data);

    $widgetGrouper = new WidgetGrouper($collection);
    $expectedCount = 7;
    $testCount = $widgetGrouper->getCountByColor('black');

    $this->assertTrue(
        count($collection) > 8,
        "Did not have at least 8 widgets in our collection"
    );
    $this->assertEquals(
        $expectedCount,
        $testCount,
        "Did not get expected count of black widgets"
    );

## Fake It When You Need To 

Remember I pointed out earlier in the chapter that it's good
to have realistic testing data? When you write tests, it's very
tempting to take short cuts. For instance, I have been
known to overuse such famous people as Testy McTesterton and
Art Vandelay as test subjects.

[Faker](https://github.com/fzaninotto/Faker) is a great tool for randomly
generating data like names, addresses, and phone numbers.

{ lang: php }
    <?php
    $faker = Faker\Factory::create();
    $foo = new Foo();
    $foo->name = $faker->name;
    $foo->address1 = $faker->streetAddress;
    $foo->address2 = null;
    $foo->city = $faker->city;
    $foo->state = $faker->state;
    $foo->zip = $faker->postcode;

    // $dbh is our PDO database handler
    $fooMapper = new FooMapper($dbh);
    $fooMapper->create($foo);

Faker has a ridiculous number of options available to you, too many to
list here. A few highlights:

* localization abilities
* different types of emails
* date and time values
* user agents
* ORM integration (Propel, Doctrine)
* create your own data providers

