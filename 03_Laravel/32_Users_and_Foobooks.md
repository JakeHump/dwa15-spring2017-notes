## Preface: Bonus material
Integrating users into A4 is optional and this information will *not* be on the lecture or A4 quiz.



## Integrating your existing data and authentication
Couple different approaches:

1. *One to Many*: Every book is connected to a single user
2. *Many to Many*: Individual books aren't associated with any one user. Instead users can favorite books, powered by a many to many relationship between books and users.

Let's go with #1.


## Connect books and users
To do this, we'll need a migration to update our `books` table to have the `user_id` foreign key. This is the same thing we did when connecting authors and books.

Create the migration:
```bash
$ php artisan make:migration connect_books_and_users
```

Fill in the migration:
```php
public function up()
{
    Schema::table('books', function (Blueprint $table) {

        # Add a new INT field called `user_id` that has to be unsigned (i.e. positive)
        $table->integer('user_id')->unsigned();

        # This field `user_id` is a foreign key that connects to the `id` field in the `authors` table
        $table->foreign('user_id')->references('id')->on('users');

    });
}

public function down()
{
    Schema::table('books', function (Blueprint $table) {

        # ref: http://laravel.com/docs/5.1/migrations#dropping-indexes
        $table->dropForeign('books_user_id_foreign');

        $table->dropColumn('user_id');
    });
}
```


## Update seeders
Because we've created a foreign key between `books` and `users`, make sure your UsersTableSeeder is invoked *before* the BooksTableSeeder:

```php
# DatabaseSeeder.php
$this->call(UsersTableSeeder::class);
$this->call(TagsTableSeeder::class);
$this->call(AuthorsTableSeeder::class);
$this->call(BooksTableSeeder::class);
$this->call(BookTagTableSeeder::class);
```

Finally, update the BooksTableSeeder so that each book is associated with a user. For our example, we'll associate every book to user id 1 (`jill@harvard.edu`).

```php
Book::insert([
    'created_at' => $timestampForThisBook,
    'updated_at' => $timestampForThisBook,
    'title' => $title,
    'author_id' => $author_id,
    'published' => $book['published'],
    'cover' => $book['cover'],
    'purchase_link' => $book['purchase_link'],
    'user_id' => 1, # <--- NEW LINE
]);
```



## Update models
Next, update the `Book` and `User` model so they're aware of this new relationship.

Add this to `Book.php`:
```php
public function user() {
    return $this->belongsTo('App\User');
}
```

Add this to `User.php`:
```php
public function books() {
    return $this->hasMany('App\Book');
}
```


## Your Books
Setup complete! Now let's make it so that when a user is logged in they only see *their* books.
Update the `index` method in the BookController like so:

```php
public function index(Request $request) {

    $user = $request->user();

    # Note: We're getting the user from the request, but you can also get it like this:
    //$user = Auth::user();

    if($user) {
        # Approach 1)
        //$books = Book::where('user_id', '=', $user->id)->orderBy('title')->get();

        # Approach 2) Take advantage of Model relationships
        $books = $user->books()->orderBy('title')->get();

        # Get 3 most recently added books
        #$newBooks = Book::orderBy('created_at', 'descending')->limit(3)->get(); # Query DB
        $newBooks = $books->sortByDesc('created_at')->take(3); # Query existing Collection

    }
    else {
        $books = [];
        $newBooks = [];
    }

    return view('books.index')->with([
        'books' => $books,
        'newBooks' => $newBooks,
    ]);

}
```


### View modifications
With these changes, we also made the following changes to the book index view:

+ Changed the heading on the book index from saying *All Books* to something like *Your Books*.
+ Only show the new books call-out if there are new books.
+ If there are no books to show, link to the page to add a new book.

Study [/resources/books/index.blade.php](https://github.com/susanBuck/foobooks/blob/master/resources/views/books/index.blade.php) to see the details of these changes.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-no-books@2x.png' style='max-width:678px;' alt=''>


### Test it out
+ Login as Jill and you should see all the books (since they're all seeded to her)
+ Login as Jamal to see the &ldquo;blank slate&rdquo; page with no books to show.



## Associating books with the logged in user
Update BookController's `storeNewBook` method to set the book's user_id right before the book is saved:

```php
$book->user_id = $request->user()->id; # <--- NEW LINE
$book->save();
```

Once a book is created it is tied to that user; because of this there's no need to do anything with the *Edit Book* in regards to user associations.


## Route adjustments for guests vs. users
Now that books are associated with specific users, it doesn't make sense that guests would be able to see an index of all the books. Instead, guests should see some sort of welcome page, while users should be directed to `/books`

To handle this, we'll configure `WelcomeController@__invoke` accordingly:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Auth;

class WelcomeController extends Controller
{

    /**
	* GET
    * /
	*/
    public function __invoke() {

        if(Auth::user()) {
            return redirect('/books');
        }

        return view('welcome');
    }
}
```

Update `/resources/views/welcome.blade.php` to:
```
@extends('layouts.master')

@push('head')
    <link href='/css/books.css' rel='stylesheet'>
@endpush

@section('title')
    Foobooks
@endsection

@section('content')

	<h1>Welcome!</h1>
    Welcome to Foobooks, a personal book organizer.
    To get started <a href='/login'>login</a> or <a href='/register'>register</a>.

@endsection
```

Then update your main route (`/`) to look like this:

```php
Route::get('/', 'WelcomeController');
```

And lock down the `/books` route with auth middleware:
```php
Route::get('/books', 'BookController@index')->middleware('auth');
```

__Try it out...__
+ While logged out, visit `http://localhost/`&mdash; you should see a welcome page.
+ While logged out, try and visit `http://localhost/books`&mdash; you should be directed to the login page.
+ While logged in, visit `http://localhost/`&mdash; you should see the book listing page.
