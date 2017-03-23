# Forms with GET

## Preface
What we know about forms so far:

+ Forms are constructed using HTML elements
+ Data entered into forms can be sent to the server using the HTTP methods GET or POST
+ This data can then be extracted from PHP superglobals, $_GET or $_POST
+ Forms should be validated to ensure the visitor is entering the appropriate data
+ Data collected from a visitor via a form should be escaped before being output to the page

<br>
All these points hold true when it comes to working with forms in a framework, but Laravel will provide some tools that make our interactions with forms more robust.



## Search form
To demonstrate forms in Laravel, we're going to replicate a feature we built in our pre-Laravel iteration of Foobooks&mdash; **the ability to search for a book.**

In the following steps, I'll provide the necessary code, which will be explained in more detail in lecture.


### Step 1) Data
First step, we'll need some book data to work with, so put the following `books.json` file in your application's `database` directory. It's not *really* a database, but it is a file that holds our data, so this seems like a logical location.

```json
{
    "The Great Gatsby": {
        "id" : 0,
        "author": "F. Scott Fitzgerald",
        "tags" : ["novel","fiction","classic","wealth"],
        "published" : 1925,
        "cover" : "http://img2.imagesbn.com/p/9780743273565_p0_v4_s114x166.JPG",
        "purchase_link" : "http://www.barnesandnoble.com/w/the-great-gatsby-francis-scott-fitzgerald/1116668135?ean=9780743273565"
    },
    "The Bell Jar": {
        "id" : 1,
        "author": "Sylvia Plath",
        "tags" : ["novel","fiction","classic","women"],
        "published" : 1963,
        "cover" : "http://img1.imagesbn.com/p/9780061148514_p0_v2_s114x166.JPG",
        "purchase_link" : "http://www.barnesandnoble.com/w/bell-jar-sylvia-plath/1100550703?ean=9780061148514"
    },
    "I Know Why the Caged Bird Sings": {
        "id" : 2,
        "author": "Maya Angelou",
        "tags" : ["autobiography","nonfiction","classic","women"],
        "published" : 1969,
        "cover" : "http://img1.imagesbn.com/p/9780345514400_p0_v1_s114x166.JPG",
        "purchase_link" : "http://www.barnesandnoble.com/w/i-know-why-the-caged-bird-sings-maya-angelou/1100392955?ean=9780345514400"
    }
}
```


### Step 2) View
Next, create a new View that will display this form:
```html
{{-- /resources/views/books/search.blade.php --}}
@extends('layouts.master')

@section('title')
    Search
@endsection

@section('content')
    <h1>Search</h1>

    <form method='GET' action='/search'>

        <label for='searchTerm'>Search by title:</label>
        <input type='text' name='searchTerm' id='searchTerm' value='{{ $searchTerm or '' }}'>

        <input type='checkbox' name='caseSensitive' {{ ($caseSensitive) ? 'CHECKED' : '' }} >
        <label>case sensitive</label>

        <br>
        <input type='submit' class='btn btn-primary btn-small'>

    </form>
@endsection
```

### Step 3) Controller action
Then create a new method in the BookController that will display this view:
```php
# app/Http/Controllers/BookController.php

    /**
	* GET
    * /search
	*/
    public function search() {
        return view('books.search');
    }
```


### Step 4) Route
And finally, a route to point to this this method:

```php
# /routes/web.php
Route::get('/search', 'BookController@search');
```



## Test it out
Studying the form construction....

```html
<form method='GET' action='/search'>
    [...etc...]
</form>
```

...we can see the form is configured to submit with GET (method) via the route `/search` (action), the same route that is displaying the form.

Thus, if you enter a title (e.g. *the great gatsby*), hit submit, it should lead you to a URL like `http://foobooks.loc/search?searchTerm=the+great+gatsby`.

If it does, it means everything is set up to display and submit the form.

## Accessing form data via the Request facade
In the `BookController@search` method, we could access the form data using PHP's superglobal $_GET, but instead we're going to take advantage of Laravel's [Request](https://laravel.com/docs/5.4/requests) facade which will give us everything $_GET gives us, plus a lot more.

To explore this facade replace your `BookController@search` method with the code below which has two main changes:

1. Added the parameter `Request $request` (this is called dependency injection<sup>1</sup>)
2. Added a series of dump statements to see all the things we can do with the $request object

```php
/**
* GET
* /search
*/
public function search(Request $request) {

    # ======== Temporary code to explore $request ==========

    # See all the properties and methods available in the $request object
    dump($request);

    # See just the form data from the $request object
    dump($request->all());

    # See just the form data for a specific input, in this case a text input
    dump($request->input('searchTerm'));

    # See what the form data looks like for a checkbox
    dump($request->input('caseSensitive'));

    # Boolean to see if the request contains data for a particular field
    dump($request->has('searchTerm')) # Should be true
    dump($request->has('publishedYear')) # There's no publishedYear input, so this should be false

    # You can get more information about a request than just the data of the form, for example...
    dump($request->fullUrl());
    dump($request->method());
    dump($request->isMethod('post'));

    # ======== End exploration of $request ==========

    # Return the view with some placeholder data we'll flesh out in a later step
    return view('books.search')->with([
        'searchTerm' => '',
        'caseSensitive' => false,
        'searchResults' => []
    ]);
}
```


After updating the `search` method, refresh your form, enter some data and hit Submit. Study the code and results to learn about some of the features of the Request object.

To see a full list of methods available to Request, refer to the [API documentation](https://laravel.com/api/5.4/Illuminate/Http/Request.html) and the [docs on Requests](https://laravel.com/docs/5.4/requests#accessing-the-request).



## Process the request
At this point we're able to submit the form and get the data from the form; the next step is handling that data.

This is where procedures are going to vary depending on what your application is trying to accomplish. You'll have to come up with the specific design for what *your* application needs to do, but for the sake of an example, the following is the code for how a foobooks search could work. You'll note a lot of this code is familiar, because it's the same logic we used earlier in the semester when building foobooks v1.0, pre-Laravel.

Read carefully through the comments, as it explains what is going on and also offers some additional tips.

```php
/**
* GET
* /search
*/
public function search(Request $request) {

    # Start with an empty array of search results; books that
    # match our search query will get added to this array
    $searchResults = [];

    # Store the searchTerm in a variable for easy access
    # The second parameter (null) is what the variable
    # will be set to *if* searchTerm is not in the request.
    $searchTerm = $request->input('searchTerm', null);

    # Only try and search *if* there's a searchTerm
    if($searchTerm) {

        # Open the books.json data file
        # database_path() is a Laravel helper to get the path to the database folder
        # See https://laravel.com/docs/5.4/helpers for other path related helpers
        $booksRawData = file_get_contents(database_path().'/books.json');

        # Decode the book JSON data into an array
        # Nothing fancy here; just a built in PHP method
        $books = json_decode($booksRawData, true);

        # Loop through all the book data, looking for matches
        # This code was taken from v1 of foobooks we built earlier in the semester
        foreach($books as $title => $book) {

            # Case sensitive boolean check for a match
            if($request->has('caseSensitive')) {
                $match = $title == $searchTerm;
            }
            # Case insensitive boolean check for a match
            else {
                $match = strtolower($title) == strtolower($searchTerm);
            }

            # If it was a match, add it to our results
            if($match) {
                $searchResults[$title] = $book;
            }

        }
    }

    # Return the view, with the searchTerm *and* searchResults (if any)
    return view('books.search')->with([
        'searchTerm' => $searchTerm,
        'caseSensitive' => $request->has('caseSensitive'),
        'searchResults' => $searchResults
    ]);
}
```

The final piece of the puzzle is to display the results of the request in the view.

In `search.blade.php`, right after the form, add this code:

```html
@if($searchTerm != null)
    <h2>Results for query: <em>{{ $searchTerm }}</em></h2>

    @if(count($searchResults) == 0)
        No matches found.
    @else
        <div class='book'>
            @foreach($searchResults as $title => $book)

                <h3>{{ $title }}</h3>
                <h4>by {{ $book['author'] }}</h4>
                <img src='{{$book['cover']}}'>

            @endforeach
        </div>
    @endif
@endif
```



## Finishing touches: Retain data in inputs
To retain the data in your form inputs after submission, update the `value` attribute of your `searchTerm` input to look like this:
```html
<input type='text' name='searchTerm' id='searchTerm' value='{{ $searchTerm or '' }}'>
```

And update your `caseSensitive` checkbox to look like this:
```html
<input type='checkbox' name='caseSensitive' {{ ($caseSensitive) ? 'CHECKED' : '' }} >
```




<br><br>

## Footnotes


__<sup>1</sup>Dependency injection (Request $request)__

The practice of passing `Request $request` to a Controller method is an example of __dependency injection__.

```php
public function search(Request $request) {

    [...]

}
```

Dependency injection is a programming practice that lets you provide (i.e. inject) class dependencies on a method by method basis. The benefit of doing this is you can easily swap out different dependencies or &ldquo;mock&rdquo; dependencies for testing purposes.

Dependency injection is a more advanced topic, and not something you need to master right now. It's mentioned here just so you know why the `Request $request` object was passed to the method in the way that it was.
