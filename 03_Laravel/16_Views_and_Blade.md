# Views and Blade

## What are Views?

> &ldquo;*Views contain the HTML served by your application and separate your controller/application logic from your presentation logic.*&rdquo; -[ref](https://laravel.com/docs/views)

+ Laravel uses the [Blade templating language](http://laravel.com/docs/blade).
+ View files are stored in in `/resources/views/`.
+ Files end with the `.blade.php` extension.



## Basic example
Let's create a view for the book's show route:

```php
Route::get('/books/{title}', 'BookController@show');
```


Via the `show` method in `BookController.php` which currently looks like this:
```php
/**
* GET
* /books/{title?}
*/
public function show($title = null) {
    return 'Show the book '.$title;
}
```

Our goal is to replace that `return 'Show the book '.$title;` with a View file.



## Create your first view file
Start by creating a new, blank file in `resources/views/books/show.blade.php`.

(The `books` directory does not yet exist in `resources/views/`... you should create this when creating this file.)

Note how the file name `show.blade.php` ends in `.blade.php`. This is required in order to use the Blade templating engine.

__Organizing View files:__ How you organize your view files is up to you, but one suggested approach...

+ Create a subfolder for each Controller. I.e. in this case we have a subfolder `/resources/views/books/` for the `BookController.php`.
+ Make the Controller's method name (`show`) correspond to the file name (`show.blade.php`).

The benefit of these two conventions is it makes it easy to correlate controller actions to view files.



## Build your first View
With your new view file created, fill it with the following content:

```html
<!DOCTYPE html>
<html>
<head>
	<title>Show Book</title>
	<meta charset='utf-8'>
    <link href="/css/foobooks.css" type='text/css' rel='stylesheet'>
</head>
<body>

	<header>
		<img
        src='http://making-the-internet.s3.amazonaws.com/laravel-foobooks-logo@2x.png'
        style='width:300px'
        alt='Foobooks Logo'>
	</header>

	<section>
	    <h1>Show book: {{ $title }}</h1>
	</section>

	<footer>
		&copy; {{ date('Y') }}
	</footer>

    <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js"></script>

</body>
</html>
```

Note that most of the above file looks like regular HTML. The exception is the content in the `<section>` and `<footer>` where you see the double brackets being used (`{{ }}`). Those brackets are [Blade](https://laravel.com/docs/blade) syntax, and they are short for &ldquo;echo this to the page&rdquo;.

So when you see something like this...
```php
{{ $title }}
```

...it's just a Blade shortcut for this PHP code:
```php
<?php echo $title; ?>
```



## Use your first View
With your first View file built, the next step is to have the Controller load this View:

```php
    
    /**
    * GET
    * /books/{title?}
    */
    public function show($title) {
        return view('books.show')->with(['title' => $title]);
    }
```

Observations about the above code:
+ Views can be loaded with the global helper `view()`.
+ Omit the `blade.php` extension since it's assumed. I.e. `show.blade.php` becomes just `show`.
+ The view file name is specified using dot notation. I.e. instead of writing `books/show` you write `books.show`.
+ You don't have to include `resources/views/` as part of the path to your view&mdash; it's assumed.
+ The `with()` method is used to pass data to the view, in this case an array of key => value pairs is passed.
+ That data is echo'd out in the view using this Blade syntax: `{{ $title }}`.

With the above code in place, test out your new view by visiting `http://foobooks.loc/books/war-and-peace` in your browser.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-first-view-loaded@2x.png' style='max-width:508px; width:100%' alt=''>



## Template inheritance
If we continued down the path outlined above, we'd create individual Views for all of our Book routes. The problem with this is you'd end up with repeat code as each view would include your doctype, the `<head>`, the `<header>` with the logo, the `<footer>`, etc.

A better solution is to divide your views into __layout templates__ and __child views__.

This is called __template inheritance__ and a simple example is conveyed in this graphic:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-view-inheritance@2x.png' class='' style='max-width:773px; width:100%' alt=''>

The idea behind template inheritance is you create a __master layout__ view (blue diagram above) that contains common, shared parts that similar HTML files/views will need. For example, the doctype, the `<head>`, CSS includes, JS includes, menus, footers, etc. In your master layout, you'll define sections where content that's specific to different pages will go.

Then, you'll create __child__ views for individual pages (red diagrams above) that __inherit__ the master template and define specific content.



## Template inheritance example
Let's apply template inheritance to our book example thus far and start by creating a new layout view at `/resources/views/layouts/master.blade.php` (note `layouts` is a new subdirectory you'll use to organize all your layout Views.)

In this new file, add these contents:

```html
<!DOCTYPE html>
<html>
<head>
	<title>
        @yield('title', 'Foobooks')
    </title>

	<meta charset='utf-8'>
    <link href="/css/foobooks.css" type='text/css' rel='stylesheet'>

    @stack('head')

</head>
<body>

	<header>
		<img
        src='http://making-the-internet.s3.amazonaws.com/laravel-foobooks-logo@2x.png'
        style='width:300px'
        alt='Foobooks Logo'>
	</header>

	<section>
		@yield('content')
	</section>

	<footer>
		&copy; {{ date('Y') }}
	</footer>

	<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>

    @stack('body')

</body>
</html>
```

Note how everything in this master layout is pretty generic&mdash; it could be applied to any page of our Foobooks app thus far.

Next, *replace* the content in `/resources/views/books/show.blade.php` so it contains just this:

```html
@extends('layouts.master')


@section('title')
    Show book
@endsection


@push('head')
    <link href="/css/books/show.css" type='text/css' rel='stylesheet'>
@endpush


@section('content')
    @if($title)
        <h1>Show book: {{ $title }}</h1>
    @else
        <h1>No book chosen</h1>
    @endif
@endsection


@push('body')
    <script src="/js/books/show.js"></script>
@endpush

```

Run `books/war-and-peace` in your browser and test everything is working.

It should look exactly as it did before:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-first-view-loaded@2x.png' style='max-width:508px; width:100%' alt=''>


## Required readings
From: [Laravel Docs: Blade](https://laravel.com/docs/5.4/blade)
+ [#introduction](https://laravel.com/docs/5.4/blade#introduction)
+ [#template-inheritance](https://laravel.com/docs/5.4/blade#template-inheritance)
+ [#displaying-data](https://laravel.com/docs/5.4/blade#displaying-data)
+ [#control-structures](https://laravel.com/docs/5.4/blade#control-structures)
+ [#including-sub-views](https://laravel.com/docs/5.4/blade#including-sub-views)
+ [#stacks](https://laravel.com/docs/5.4/blade#stacks)
