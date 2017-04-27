## Database Relationships
A web application typically contains multiple database tables, and those tables often relate to one another in some way.

For insight into this, consider the following Foobooks database schema, outlining the table design we'll be working towards.

Each blue arrow represents a relationship between two tables.
<img src='http://making-the-internet.s3.amazonaws.com/laravel-foobooks-schema@2x.png' style='max-width:875px;' alt=''>

(Diagrams created with <https://dbdesigner.net>)

[Example of a more complex database design...](http://making-the-internet.s3.amazonaws.com/complex-database-design@2x.png)

## Relationship Types
There are 6 common relationship types:

+ One To One
+ One To Many
+ Many To Many
+ Has Many Through
+ Polymorphic Relations
+ Many To Many Polymorphic Relations

You can read about all the relationships here: [Docs: Eloquent Relationships](http://laravel.com/docs/eloquent-relationships).

In these notes, we'll start with the __One To Many__ relationship.




## Authors
Continuing with the Foobooks example, we can expand on our database design to include an `authors` table.

This addition introduces the following relationship to our database, [One to Many](http://laravel.com/docs/eloquent#one-to-many):

+ Author *has many* Books
+ Books *belongs to* Author

Example:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-one-to-many@2x.png' style='max-width:644px; width:100%' alt='One To Many Relationship'>

The database design necessary for this relationship looks like this:
<img src='http://making-the-internet.s3.amazonaws.com/laravel-one-to-many-schema@2x.png' style='max-width:314px;' alt=''>



## Create the authors table
First we need to create, build, and run the migration to create a new `authors` table.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-authors-table@2x.png' style='max-width:px;' alt=''>


Create the migration:
```bash
$ php artisan make:migration create_authors_table
```

Fill in the up/down methods:
```php
public function up()
{
    Schema::create('authors', function (Blueprint $table) {

        $table->increments('id');
        $table->timestamps();

        $table->string('first_name');
        $table->string('last_name');
        $table->integer('birth_year');
        $table->string('bio_url');

    });
}

public function down()
{
    Schema::drop('authors');
}
```


Generate an `Author` model:
```bash
$ php artisan make:model Author
```



## Seed the authors table

Create the seeder:
```bash
$ php artisan make:seeder AuthorsTableSeeder
```

Make the Author model available at the top of this new seeder file (`/seeds/AuthorsTableSeeder.php`):
```
use App\Author;
```

Fill in the run method:
```php

public function run()
{
    # Array of author data to add
    $authors = [
        ['F. Scott', 'Fitzgerald', 1896, 'https://en.wikipedia.org/wiki/F._Scott_Fitzgerald'],
        ['Sylvia', 'Plath', 1932, 'https://en.wikipedia.org/wiki/Sylvia_Plath'],
        ['Maya', 'Angelou', 1928, 'https://en.wikipedia.org/wiki/Maya_Angelou'],
        ['J.K.', 'Rowling', 1965, 'https://en.wikipedia.org/wiki/J._K._Rowling']
    ];

    # Initiate a new timestamp we can use for created_at/updated_at fields
    $timestamp = Carbon\Carbon::now()->subDays(count($authors));

    # Loop through each author, adding them to the database
    foreach($authors as $author) {

        # Set the created_at/updated_at for each book to be one day less than
        # the book before. That way each book will have unique timestamps.
        $timestampForThisAuthor = $timestamp->addDay()->toDateTimeString();
        Author::insert([
            'created_at' => $timestampForThisAuthor,
            'updated_at' => $timestampForThisAuthor,
            'first_name' => $author[0],
            'last_name' => $author[1],
            'birth_year' => $author[2],
            'bio_url' => $author[3],
        ]);
    }
}

}
```

Update the `run` method in `/database/seeds/DatabaseSeeder.php` to invoke this new seeder; because books will now be associated with authors, the author's table needs to be seeded first.

```php
public function run()
{
	# Because `books` will be associated with `authors`,
    # authors should be seeded first
	$this->call(AuthorsTableSeeder::class);
	$this->call(BooksTableSeeder::class);
}
```




## Connect authors and books tables
Now that we have an `authors` table, we need to modify the `books` table adding a __foreign key field__ that will connect each book to a row in the `authors` table.

__Naming convention for foreign key fields:__ singularize the table name you're connecting to and append `_id`. In this case, that means the foreign key field name is `author_id`.

We could bundle this change into the original `create_books_table` migration, but it can be nice to have stand-alone migrations for connecting tables. This way you can look at your migrations and see at a glance where the connections are.

And so, we'll create a new migration:
```bash
$ php artisan make:migration connect_authors_and_books
```

Fill in the up/down methods. Note that foreign keys have to be __unsigned__ (i.e. positive).

```php

public function up()
{
    Schema::table('books', function (Blueprint $table) {

        # Remove the field associated with the old way we were storing authors
        # Can do this here, or update the original migration that creates the `books` table
        # $table->dropColumn('author');

		# Add a new INT field called `author_id` that has to be unsigned (i.e. positive)
        $table->integer('author_id')->unsigned();

		# This field `author_id` is a foreign key that connects to the `id` field in the `authors` table
        $table->foreign('author_id')->references('id')->on('authors');

    });
}

public function down()
{
    Schema::table('books', function (Blueprint $table) {

        # ref: http://laravel.com/docs/migrations#dropping-indexes
        # combine tablename + fk field name + the word "foreign"
        $table->dropForeign('books_author_id_foreign');

        $table->dropColumn('author_id');
    });
}
```




## Update the books table seeder
Now that we have an authors table and seeder, we should modify the existing `BooksTableSeeder` to connect each book to the appropriate author:

```php
public function run()
{

    # [...Redacted code that sets up the $books array]

    foreach($books as $title => $book) {

        # First, figure out the id of the author we want to associate with this book

        # Extract just the last name from the book data...
        # F. Scott Fitzgerald => ['F.', 'Scott', 'Fitzgerald'] => 'Fitzgerald'
        $name = explode(' ', $book['author']);
        $lastName = array_pop($name);

        # Find that author in the authors table
        $author_id = Author::where('last_name', '=', $lastName)->pluck('id')->first();

        Book::insert([
            'created_at' => $timestampForThisBook,
            'updated_at' => $timestampForThisBook,
            'title' => $title,
            #'author' => $book['author'], # Remove the old way we stored the author
            'author_id' => $author_id, # Add the new way we store the author
            'published' => $book['published'],
            'cover' => $book['cover'],
            'purchase_link' => $book['purchase_link'],
        ]);

    }
}
```




## Migrate and Seed
If the above steps were executed correctly, you should be able to run the following command to rebuild and seed your tables:

```bash
$ php artisan migrate:refresh --seed
```

If you have any issues with the above steps, revisit the notes on Migrations and Seeding.

__This concludes the table setup for this relationship. Next, we'll look at how we can tell our Models about these relationships.__




## Identifying Relationships in Models
Relationships between tables need to be defined in their corresponding Models via __relationship methods__.

For example, the Author model should have a method called `books()` which returns the Eloquent relationship [hasMany](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Model.html#method_hasMany):


```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Author extends Model

	public function books() {
		# Author has many Books
		# Define a one-to-many relationship.
		return $this->hasMany('App\Book');
	}
}
```

On the flip side, the Book class should have a method called `author()` which returns the Eloquent relationship [belongsTo](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Model.html#method_belongsTo):

```php
<?php

namespace App;

use Illuminate\Database\Eloquent\Model;

class Book extends Model {

	public function author() {
		# Book belongs to Author
		# Define an inverse one-to-many relationship.
		return $this->belongsTo('App\Author');
	}
}
```

Now that we've made these Models &ldquo;aware&rdquo; of these relationships, we can start using them...




## Associate an author with a book
Here's an example where we create a book, and then associate that book with an author.


```php
# To do this, we'll first create a new author:
$author = new Author;
$author->first_name = 'J.K';
$author->last_name = 'Rowling';
$author->bio_url = 'https://en.wikipedia.org/wiki/J._K._Rowling';
$author->birth_year = '1965';
$author->save();
dump($author->toArray());

# Then we'll create a new book and associate it with the author:
$book = new Book;
$book->title = "Harry Potter and the Philosopher's Stone";
$book->published = 1997;
$book->cover = 'http://prodimage.images-bn.com/pimages/9781582348254_p0_v1_s118x184.jpg';
$book->purchase_link = 'http://www.barnesandnoble.com/w/harrius-potter-et-philosophi-lapis-j-k-rowling/1102662272?ean=9781582348254';
$book->author()->associate($author); # <--- Associate the author with this book
$book->save();
dump($book->toArray());
```

Note how the `associate()` method is called *before* the `save()` method. This is because `associate` is setting the `author_id` field on the books row; that setting should happen before the row is created.

Another way to look at the `associate()` method is that it's doing this:

```php
$book->author_id = $author->id;
```





## Querying with relationships
Once your Models have been programmed with relationships, it's easy to join data amongst multiple tables using [dynamic properties](http://laravel.com/docs/eloquent#dynamic-properties).

For example, you can fetch a book and have access to its author info...

```php
# Get the first book as an example
$book = Book::first();

# Get the author from this book using the "author" dynamic property
# "author" corresponds to the the relationship method defined in the Book model
$author = $book->author;

# Output
dump($book->title.' was written by '.$author->first_name.' '.$author->last_name);
dump($book->toArray());
```



## Eager Loading
If you're querying for many books, you may want to join in the related author data with that query. This can be done via the [with()](http://devdocs.io/laravel/api/4.2/illuminate/database/eloquent/model#method_with) method and is referred to as **eager loading**:

```php
# Eager load the author with the book
$books = Book::with('author')->get();

foreach($books as $book) {
    echo $book->author->first_name.' '.$book->author->last_name.' wrote '.$book->title.'<br>';
}

dump($books->toArray());

```




## Tips
If you receive an error like this:

```xml
General error: 1005 Can't create table 'database-name.#sql-13993_88'
(errno: 150) (SQL: alter table `books` add constraint books_author_id_foreign foreign key
(`author_id`) references `authors` (`id`))`                                                                      
```

There may be two possible causes:

__Cause 1)__ When building relationships, it's important that you create tables in a logical order. For example: if the table `books` will have a FK connecting it to `authors`, then the `authors` table should exist before attempting to create that FK.

__Cause 2)__ When creating a column that will contain a FK relating it to another table, make sure that column is unsigned. Example:

```php
# Important! FK has to be unsigned because the PK it will reference is auto-incrementing
$table->integer('author_id')->unsigned();
```



## Reference
+ [Docs: Relationships](http://laravel.com/docs/eloquent#relationships)
+ [Docs: Relationships: One to Many](http://laravel.com/docs/eloquent-relationships#one-to-many)
+ [Docs: Migrations: Foreign Key Constraints](http://laravel.com/docs/migrations#foreign-key-constraints)
