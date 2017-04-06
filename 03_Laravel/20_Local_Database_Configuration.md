# Local Database Configuration

## Starting assumptions
+ You've already created a database, per the instructions in the *Databases Primer* notes. Our example database name is `foobooks`.
+ MAMP or XAMPP is open and your MySQL database server is running.

<img src='http://making-the-internet.s3.amazonaws.com/mysql-running-in-xampp-and-mamp@2x.png' style='max-width:921px; width:100%' alt='MySQL running in XAMPP and MAMP'>






## Laravel database configuration
In order for your application to connect to your local MySQL server and database, Laravel needs your MySQL host name, username, and password as well as the name of the database you want to connect to.

These configurations are set in `/config/database.php`, so open that file.

For our examples we'll be using the MySQL database that comes with MAMP so find `mysql` in the `connections` array.

```php
'mysql' => [
    'driver' => 'mysql',
    'host' => env('DB_HOST', '127.0.0.1'),
    'port' => env('DB_PORT', '3306'),
    'database' => env('DB_DATABASE', 'forge'),
    'username' => env('DB_USERNAME', 'forge'),
    'password' => env('DB_PASSWORD', ''),
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
    'strict' => true,
    'engine' => null,
],
```

Now how the `env()` method is used, making it easy to set separate database configurations for your different environments. This is important, because you'll be using two different databases&mdash; one locally and one on production (DigitalOcean).

Open your local `/.env` file and you should see 6 `DB_` constants are set with some defaults:

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=homestead
DB_USERNAME=homestead
DB_PASSWORD=secret
```

We need to update `DB_HOST`, `DB_DATABASE`, `DB_USERNAME`, and `DB_PASSWORD` with values relavent to our local servers.


__MAMP users__, your configs will look like this:
```xml
DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=foobooks
DB_USERNAME=root
DB_PASSWORD=root
```

(FYI The values for host, username, and password are found on the MAMP start page at <http://localhost/MAMP>)


__XAMPP__ users, your configs will look like this:
```
DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=foobooks
DB_USERNAME=root
DB_PASSWORD=
```

Note that `DB_PASSWORD` is purposefully left blank.

(FYI The above values are found in `C:/xampp/passwords.txt`)




## Test your connection
Here is temporary code you can put in your web Routes file to test that the database connection is working:

```php
Route::get('/debug', function() {

	echo '<pre>';

	echo '<h1>Environment</h1>';
	echo App::environment().'</h1>';

	echo '<h1>Debugging?</h1>';
	if(config('app.debug')) echo "Yes"; else echo "No";

	echo '<h1>Database Config</h1>';
    echo 'DB defaultStringLength: '.Illuminate\Database\Schema\Builder::$defaultStringLength;
    /*
    The following commented out line will print your MySQL credentials.
    Uncomment this line only if you're facing difficulties connecting to the database and you
    need to confirm your credentials.
    When you're done debugging, comment it back out so you don't accidentally leave it
    running on your production server, making your credentials public.
    */
	//print_r(config('database.connections.mysql'));

	echo '<h1>Test Database Connection</h1>';
	try {
		$results = DB::select('SHOW DATABASES;');
		echo '<strong style="background-color:green; padding:5px;">Connection confirmed</strong>';
		echo "<br><br>Your Databases:<br><br>";
		print_r($results);
	}
	catch (Exception $e) {
		echo '<strong style="background-color:crimson; padding:5px;">Caught exception: ', $e->getMessage(), "</strong>\n";
	}

	echo '</pre>';

});
```

When you visit the resulting route (`http://localhost/debug`) you should see some debugging information, and if everything went well a message saying `Connection confirmed` followed by a list of the databases on your server.

If you see a red `Caught exception:` message, you have some troubleshooting to do.

<small>
Note: The above code is echo'ing a lot of content (including HTML) to the page directly from your Routes file,
which is typically something we want to avoid. We're making an exception here because the code is temporary for debugging purposes.
</small>




## Conclusions
With your database setup and your connection confirmed, you're ready to move on to **Migrations** in order to build your tables.
