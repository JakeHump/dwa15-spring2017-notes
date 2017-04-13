Database seeding is the process of entering sample/testing data into your tables.

Benefits of using seed data:
+ Good for __development environments__, where you don't have real data to work with yet.
+ Good for __testing__, so that your testing scripts has disposable data to work with.
+ Good for __collaboration__; if another developer (or TA or instructor) clones your project they can quickly build their tables using your migrations, and fill those tables with sample data using your seeders.

We saw a quick-and-dirty example of seeding the `books` table at the start of the Eloquent notes when you ran raw SQL in phpMyAdmin to enter some books.

This works, but a better way of managing seed data is via scripts called __seeders__.


## Create your first seeder
Let's start by creating a new seeder for the books table.

There's an artisan command `make:seeder` that can be used to generate a boilerplate seeder class:

```bash
php artisan make:seeder BooksTableSeeder
```

This will generate a new file in `/database/seeds/BooksTableSeeder.php` which includes one method, `run`.

Amend this new file adding a `use App\Book;` statement at the top so you can use the Book model to add some books.

```php
<?php

use Illuminate\Database\Seeder;

use App\Book; # <------ ADD THIS

class BooksTableSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        //
    }
}
```



The `run` method is invoked when this seeder is called. Given that, the `run` method is where you'll put the code that will actually enter seed data into the *books* table.


```php
public function run()
{
    Book::insert([
        'created_at' => Carbon\Carbon::now()->toDateTimeString(),
        'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
        'title' => 'The Great Gatsby',
        'author' => 'F. Scott Fitzgerald',
        'published' => 1925,
        'cover' => 'http://img2.imagesbn.com/p/9780743273565_p0_v4_s114x166.JPG',
        'purchase_link' => 'http://www.barnesandnoble.com/w/the-great-gatsby-francis-scott-fitzgerald/1116668135?ean=9780743273565',
    ]);

    Book::insert([
        'created_at' => Carbon\Carbon::now()->toDateTimeString(),
        'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
        'title' => 'The Bell Jar',
        'author' => 'Sylvia Plath',
        'published' => 1963,
        'cover' => 'http://img1.imagesbn.com/p/9780061148514_p0_v2_s114x166.JPG',
        'purchase_link' => 'http://www.barnesandnoble.com/w/bell-jar-sylvia-plath/1100550703?ean=9780061148514',
    ]);

    Book::insert([
        'created_at' => Carbon\Carbon::now()->toDateTimeString(),
        'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
        'title' => 'I Know Why the Caged Bird Sings',
        'author' => 'Maya Angelou',
        'published' => 1969,
        'cover' => 'http://img1.imagesbn.com/p/9780345514400_p0_v1_s114x166.JPG',
        'purchase_link' => 'http://www.barnesandnoble.com/w/i-know-why-the-caged-bird-sings-maya-angelou/1100392955?ean=9780345514400',
    ]);

    Book::insert([
        'created_at' => Carbon\Carbon::now()->toDateTimeString(),
        'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
        'title' => 'Harry Potter and the Sorcerer\'s Stone',
        'author' => 'J.K. Rowling',
        'published' => 1997,
        'cover' => 'http://prodimage.images-bn.com/pimages/9780590353427_p0_v1_s484x700.jpg',
        'purchase_link' => 'http://www.barnesandnoble.com/w/harry-potter-and-the-sorcerers-stone-j-k-rowling/1100036321?ean=9780590353427',
    ]);

    Book::insert([
        'created_at' => Carbon\Carbon::now()->toDateTimeString(),
        'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
        'title' => 'Harry Potter and the Chamber of Secrets',
        'author' => 'J.K. Rowling',
        'published' => 1998,
        'cover' => 'http://prodimage.images-bn.com/pimages/9780439064873_p0_v1_s192x300.jpg',
        'purchase_link' => 'http://www.barnesandnoble.com/w/harry-potter-and-the-chamber-of-secrets-j-k-rowling/1004338523?ean=9780439064873',
    ]);

    Book::insert([
        'created_at' => Carbon\Carbon::now()->toDateTimeString(),
        'updated_at' => Carbon\Carbon::now()->toDateTimeString(),
        'title' => 'Harry Potter and the The Prisoner of Azkaban',
        'author' => 'J.K. Rowling',
        'published' => 1999,
        'cover' => 'http://prodimage.images-bn.com/pimages/9780439136365_p0_v1_s192x300.jpg',
        'purchase_link' => 'http://www.barnesandnoble.com/w/harry-potter-and-the-prisoner-of-azkaban-j-k-rowling/1100178339?ean=9780439136365',
    ]);
}
```


__Carbon Time__

For the `created_at` and `updated_at` fields, we used the `Carbon::now()->toDateTimeString()` method to generate a new timestamp based on the current time.

Carbon is a package that comes with Laravel and provides many date/time methods; you can read more about Carbon here: <https://github.com/briannesbitt/Carbon>.

Using `Carbon::now()` as we have in this example will give all of your books the same `created_at`/ `updated_at` values. If you need your sample data to have unique timestamp fields, you could use a package like [Faker](https://github.com/fzaninotto/Faker) to generate random time strings.









## Master seeder
Next, open `database/seeds/DatabaseSeeder.php` where you should see this existing code:

```php
<?php

use Illuminate\Database\Seeder;

class DatabaseSeeder extends Seeder
{
    /**
     * Run the database seeds.
     *
     * @return void
     */
    public function run()
    {
        // $this->call(UserTableSeeder::class);
    }
}
```

This `DatabaseSeeder` class is the master seed file that gets called when you run `php artisan db:seed`. Its job is to invoke your individual Seeder classes.

There's a sample invocation commented out:

```php
//$this->call(UserTableSeeder::class);
```

Following this pattern, we can create a statement to invoke the BooksTableSeeder:

```php
$this->call(BooksTableSeeder::class);
```

Finally, run `php artisan db:seed` to run the master seeder, which should invoke the `BooksTableSeeder`.

You should see a result like this:

```
$ php artisan db:seed
Seeded: BooksTableSeeder
```

Open phpMyAdmin to view your *books* table and confirm your seed data was added.



## Running seeds again

If you call `php artisan db:seed` again, it will rerun your seeds, creating duplicate rows, which may not be what you want.

To prevent duplicate rows, you could manually empty your database tables via phpMyAdmin, then run the seeders again.

__Or__, you could run `php artisan migrate:refresh` to rebuild all your tables, effectively emptying them.
Then you can run `php artisan db:seed` again.

__Or__, you can do the last two steps in one with the `migrate:refresh` command followed by the `--seed` flag:

```bash
$ php artisan migrate:refresh --seed
```




## Seeding in Assignment 4
In addition to migrations, it's required that your Assignment 4 application utilizes database seeding to fill all tables with example data. This practice makes it quick for us to easily build and fill your database tables so that we can more efficiently help you troubleshoot.



## Read More
+ <http://laravel.com/docs/seeding>
