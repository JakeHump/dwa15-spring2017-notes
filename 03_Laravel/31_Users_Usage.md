## Preface: Bonus material
Integrating users into A4 is optional and this information will *not* be on the lecture or A4 quiz.


## Users
With the mechanisms for registering and logging in/out built, you can now adapt your application to do different things based on whether the user is logged in not.



## Auth::check()
Let's start by adapting the Foobooks navigation menu

If the visitor is *not* logged in, they should see these links:
+ Home
+ Register
+ Login

<br>
If the visitor *is* logged in they should these links:
+ Home
+ Search
+ Add a book
+ Logout

<img src='http://making-the-internet.s3.amazonaws.com/laravel-menu-for-guest-vs-user@2x.png' style='max-width:1049px; width:100%' alt=''>
<br>
To accomplish this, we'll use the __`Auth::check()`__ method which returns `True` if the user is logged in or `False` if they are not.

In `resources/views/layouts/master.blade.php` the `<nav>` will be updated to look like this:

```php
<nav>
    <ul>
        @if(Auth::check())
            <li><a href='/'>Home</a></li>
            <li><a href='/search'>Search</a></li>
            <li><a href='/books/new'>Add a book</a></li>
            <li>
                <form method='POST' id='logout' action='/logout'>
                    {{csrf_field()}}
                    <a href='#' onClick='document.getElementById("logout").submit();'>Logout</a>
                </form>
            </li>
        @else
            <li><a href='/'>Home</a></li>
            <li><a href='/login'>Login</a></li>
            <li><a href='/register'>Register</a></li>
        @endif
    </ul>
</nav>
```




## Manually restricting routes
One of the purposes of authentication is to keep non-registered users away from certain routes.

For Foobooks, we want to set it up so that any visitor can view books, but only logged in users can add books.

One way we could approach this is to update the `BookController@getCreate` method so it redirects the user away from the page if they're not logged in like so:

```php
public function createNewBook() {
    if(!Auth::check() ) {
        Session::flash('message','You have to be logged in to create a new book');
        return redirect('/');
    }
    return view('books.create');
}
```

This works, but can become tedious if you have many actions that need to be restricted. An alternative method is to use Middleware...




## Restricting routes with Middleware
Laravel has a feature called [Middleware](http://laravel.com/docs/middleware) which lets you filter requests entering your application. (Earlier, we saw an example of Middleware with the CSRF checking that happens when forms are submitted.)

Laravel ships with a default Middleware file for authentication (`Illuminate\Auth\Middleware\Authenticate`). When this Middleware filter is used, a check is done to see if a user is authenticated, and if not they're redirected to the login page.

To understand how this Middleware is used, you should open `/app/Http/Kernel.php` and note that it's part of the `$routeMiddleWare` array with the key `auth`:

```php
protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
];
```

Knowing this, we'll adapt a route to use this Middleware.

For example, here's the existing route to view the `/books/create` page:

```php
Route::get('/books/new', 'BookController@createNewBook');
```

We'll update this route to look like this:

```php
Route::get('/books/new', [
    'middleware' => 'auth',
    'uses' => 'BookController@createNewBook'
]);
```

Now, when `/books/new` is visited, the `auth` middleware will apply.

__Test it out:__

If you are not logged in and you attempt to visit `http://localhost/books/new` you should be redirected to the login page, with a flash message telling you `You must be logged in to access this page.`.

Then, if you successfully login, you'll be sent back to `http://localhost/books/new`.



## Restricting multiple routes with Middleware
In the above example, we restricted a single route, but you can also restrict many routes using a route group like this:

```php
Route::group(['middleware' => 'auth'], function () {
    Route::get('/book/create', 'BookController@getCreate');
    Route::post('/book/create', 'BookController@postCreate');

    Route::get('/book/edit/{id?}', 'BookController@getEdit');
    Route::post('/book/edit', 'BookController@postEdit');
});
```


## Restricting routes to guests only
In the above examples we restricted routes to logged in users.

On the flip side, you may also want to restrict routes so that only &ldquo;guests&rdquo; (i.e. non-logged in users) can see them.

For example, if your user is already logged in, you don't want them to be able to visit `http://localhost/login` or `http://localhost/register`

Some of this functionality is already set up for you with the baked in authentication files Laravel gives you, so let's trace what's going on...

First, look at the `LoginController.php` and note the `__construct()` method invokes a middleware `guest`:

```php
public function __construct()
{
    $this->middleware('guest', ['except' => 'logout']);
}
```

This line is dictating that every method/action in AuthController will use the `guest` middleware, *except* for `logout`.

You can test if it's working by logging in (`http://localhost/login`) and then attempting to visit `http://localhost/login` or `http://localhost/register`.

If everything is working, it should redirect you to `http://localhost/`.



## Reference
+ [Docs: Authentication](http://laravel.com/docs/authentication)
+ [Docs: Middleware](http://laravel.com/docs/middleware)
