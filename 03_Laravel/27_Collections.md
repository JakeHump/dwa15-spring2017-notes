An Eloquent query returns a __Collection object__.

The Collection object is a wrapper for an Array called **items** which contains all the data of your query *and* methods for working with this data.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-collection@2x.png' class='' style='max-width:951px; width:100%' alt='Eloquent queries return Collections'>





## Collection Magic
Collections come built in with [magic methods](http://php.net/manual/en/language.oop5.magic.php) that lets them (the Collections) adapt to how you use them.

### String magic
For example, if you treat a Collection like a String (e.g. you `echo` it), it will transform itself into a JSON string.

```php
$books = Book::all();

# This will output a JSON string
echo $books;
```

This works because the Collection class contains the [__toString](http://php.net/manual/en/language.oop5.magic.php#object.tostring) magic method which is programmed to output a JSON string.

### Array magic
You can also treat a Collection like an array:

```php
$books = Book::all();

# loop through the Collection and access just the data
foreach($books as $book) {
    echo $book['title']."<br>";
}
```

Depending on your data, the results of the above might look like this:
```text
The Great Gatsby
The Bell Jar
I Know Why the Caged Bird Sings
```

This works because the Collection class implements PHP's [IteratorAggregate](http://php.net/manual/en/class.iteratoraggregate.php) interface.


### Object magic
Or, if you prefer object notation...

```php
$books = Book::all();

foreach($books as $book) {
	echo $book->title."<br>";
}
```

Depending on your data, the results of the above might look like this:
```text
The Great Gatsby
The Bell Jar
I Know Why the Caged Bird Sings
```




## Using Collections in your Views
Because of the above points, you can pass a Collection to a View where it can be iterated through like a regular array or object:

Controller:
```php
function practiceX() {
    $books = Book::all();
    return view('books.index')->with('books', $books);
}
```

View:
```php
@foreach($books as $book)
    <h2>{{ $book->title }}</h2>
@endforeach
```




### Get *just* the data
If you want just a &ldquo;pure&rdquo; array of the data in your Collection, use can use the [toArray()](http://laravel.com/api/5.0/Illuminate/Support/Collection.html#method_toArray) method. This will "strip" out all the methods of the Collection, giving you *just* the data attributes.

```php
$books = Book::all();
dump($books->toArray());
```

Depending on your data, the results of the above might look like this:
```text
array:3 [▼
  0 => array:9 [▼
    "id" => 1
    "created_at" => "2015-11-11 04:03:26"
    "updated_at" => "2015-11-11 04:03:26"
    "title" => "The Great Gatsby"
    "author" => "F. Scott Fitzgerald"
    "page_count" => 0
    "published" => 1925
    "cover" => "http://img2.imagesbn.com/p/9780743273565_p0_v4_s114x166.JPG"
    "purchase_link" => "http://www.barnesandnoble.com/w/the-great-gatsby-francis-scott-fitzgerald/1116668135?ean=9780743273565"
  ]
  1 => array:9 [▶]
  2 => array:9 [▶]
]
```



## Collection Methods
Below is a list of some of the most commonly used methods built into the Collection object.

Refer to the [Collections: Available Methods](https://laravel.com/docs/5.2/collections#available-methods) for a full list.


| Method   |      Usage      |
|----------|-------------|
| [`all()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_all) |  Get all of the items in the collection. |
| [`first()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_first) | Get the first item from the collection. |
| [`last()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_last) | Get the last item from the collection. |
| [`shift()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_shift) | Get and remove the first item from the collection. |
| [`pop()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_pop) | Get and remove the last item from the collection. |
| [`each()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_each) | Loop through each item in a collection. Can be used as an alternative to a regular `foreach`. |
| [`map()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_map) | Loop through a collection, returning a new collection as a result. Good for copying and editing a collection.
| [`filter()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_filter) | Loop through a collection, returning true/false as you go. What is true stays, what is false gets removed.
| [`sort()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_sort) | Sort through each item with a callback.
| [`reverse()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_reverse) | Reverses a collection.
| [`isEmpty()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_isEmpty) | Determine if the collection is empty or not.
| [`toArray()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_toArray) | Get the collection of items as a plain array.
| [`toJson()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_toJson) | Get the collection of items as JSON.
| [`count()`](http://laravel.com/api/5.0/Illuminate/Database/Eloquent/Collection.html#method_count) | Count the number of items in the collection. |

Some of these method names may look familiar (e.g. `first`, `all`) because we saw them when learning about Eloquent query constraints.

So what's the difference? Read on...


## Query Responsibility
When possible, you want to minimize the number of queries made to a database, as excess queries can slow down an application's load time.

One way to minimize queries is to look for ways you can essentially re-use the data from an existing query.

__For example...__

Imagine a scenario in which you're building a page that shows an alphabetical listing of all the books in the foobooks database. You start with an Eloquent query:

```php
$books = Book::orderBy('title')->get(); # Query DB
```

Now imagine that on the same page you also want to have a special *What's New* section at the top which highlights the three most recently added books the the library.

You *could* run another Eloquent query to fetch the three most recently added books:

```php
$newBooks = Book::orderBy('created_at', 'descending')->limit(3)->get(); # Query DB
```

But this is wasteful&mdash; the information you need already exists in the `$books` Collection from the previous query, so you should extract it from there rather than making &ldquo;another trip&rdquo; to the database. And so, using some Collection methods, you can extract the information you need from the `$books` Collection:

```php
$newBooks = $books->sortByDesc('created_at')->take(3); # Query existing Collection
```

*[Discussed in lecture: Trade-off of putting *too* much responsibility on the Collection.]*

## Read More
+ <https://laravel.com/docs/collections>
+ <https://laravel.com/api/5.4/Illuminate/Database/Eloquent/Collection.html>
