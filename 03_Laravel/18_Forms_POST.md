# Forms with POST

### View
```html
{{-- /resources/views/books/new.blade.php --}}
@extends('layouts.master')

@section('title')
    New book
@endsection

@section('content')
    <h1>Add a new book</h1>

    <form method='POST' action='/books/new'>
        {{ csrf_field() }}
        <input type='text' name='title'>
        <input type='submit' value='Submit'>
    </form>

@endsection
```


### Routes
Add *before* the `/books/{title?}` route.

```php
Route::get('/books/new', 'BookController@createNewBook');
Route::post('/books/new', 'BookController@storeNewBook');
```


### Controller actions
```php
/**
* GET
* /books/new
* Display the form to add a new book
*/
public function createNewBook(Request $request) {
    return view('books.new');
}


/**
* POST
* /books/new
* Process the form for adding a new book
*/
public function storeNewBook(Request $request) {

    $title = $request->input('title');

    # Code will eventually go here to actually save this book to a database

    return redirect('/books/'.$title);
}
```

### Test it out
http://foobooks.loc/books/new




## CSRF Protection in Laravel
The above form includes this line:

```html
{{ csrf_field() }}
```

Which will translate to something like this in your HTML;

```html
<input type='hidden' name='_token' value='Qw7HOT2zXe1ouFUuNov73baXFZTz4nHdf0CyJvZe'>
```

This line is necessary when submitting forms with `POST` (or `PUT` or `DELETE`) because it guards against CSRF attacks (explained below).

If you don't have the CSRF token in your form, you'll get a `TokenMismatchException` error when trying to submit your form using the `POST`, `PUT`, or `DELETE` methods.

CSRF protection in Laravel is an example of [__Middleware__](http://laravel.com/docs/middleware#terminable-middleware):

> *&ldquo;HTTP middleware provide a convenient mechanism for filtering HTTP requests entering your application. For example, Laravel includes a middleware that verifies the user of your application is authenticated. If the user is not authenticated, the middleware will redirect the user to the login screen. However, if the user is authenticated, the middleware will allow the request to proceed further into the application.*

> *Of course, additional middleware can be written to perform a variety of tasks besides authentication. A CORS middleware might be responsible for adding the proper headers to all responses leaving your application. A logging middleware might log all incoming requests to your application.*

> *There are several middleware included in the Laravel framework, including middleware for maintenance, authentication, CSRF protection, and more. All of these middleware are located in the app/Http/Middleware directory.&rdquo;*




## What is CSRF?
CSRF, or __Cross-Site Request Forgery__, is a type of web application vulnerability where a user unintentionally runs a script in their browser that takes advantage of their logged in status on a target site.

For example, imagine you set up a route so that when users go to `http://yourdomain.com/users/delete` it will delete their account.

You've implemented proper JavaScript checking that confirms with the user they wish to delete their account before ever triggering the `/users/delete` method.

However, a hacker has dug through your source code to find this URL and wishes to trick your users into deleting their accounts.

To accomplish this, they could present the victim with an image (in a forum post, in a email, etc.) with the delete URL set as the image src, which would execute `http://yourdomain.com/users/delete`:

```html
<img src='http://yourdomain.com/users/delete'>
```

Alternatively, they could trick you into clicking on a link leading directly to the delete page:

```html
<a href='http://yourdomain.com/users/delete'>Free stuff!</a>
```

In the above examples, the hacker is taking advantage of the fact that the victim user is potentially already logged in on the victim site.

To prevent CSRF attacks you want to verify that the origin of requests on your site are coming from within your site. This is done by sending a unique, encrypted token with each form submission, the CSRF token.

When the form submission is received by your application, Laravel automatically checks the token to make sure it's valid, thereby verifying the form submissions is legitimate.


