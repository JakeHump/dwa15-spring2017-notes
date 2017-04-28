## Preface: Bonus material
Integrating users into A4 is optional and this information will *not* be on the lecture or A4 quiz.


## Users
Typically, any application that has users needs the following components:

+ Registration page
+ Login page
+ Logout functionality
+ Forgot password page
+ Ability to &ldquo;recognize&rdquo; the user
    + Lock down pages/features depending on whether visitor is User/Guest
    + Display different content depending on whether visitor is User/Guest and which User
    + Etc...

Much of what you need for managing users on a application comes built into Laravel. That being said, there is some work you need to do to get things working.

Let's start with what exists...



## Auth Config
Open `config/auth.php` to see the default configurations for how authentication works. For our purposes, we don't have to change anything here, but you should read though the well-commented options.




## User Model
Next, there's the existing `app\User.php` which is a Eloquent model for interacting with the `users` table. You don't have to change anything here to get started.




## Migration for the `users` table
The default install of Laravel comes with a Migration file to generate the `users` table.

If you open `database/migrations/2014_10_12_000000_create_users_table.php` you can see the structure of this table by examining the `up()` method:

```php
public function up()
{
    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name');
        $table->string('email')->unique();
        $table->string('password');
        $table->rememberToken();
        $table->timestamps();
    });
}
```

This migration includes the default fields Laravel will require to have authentication work &ldquo;out of the box&rdquo;.

You can/should eventually modify this migration to include the fields needed for users in your application. For example, you may want to add additional fields (e.g. `postalCode`, `ipAddress`, `country`, etc.). Or you may want to edit existing fields like breaking out the `name` field into `firstName` and `lastName`. For now though, don't make any edits&mdash; let's just get authentication working.

Make sure your migrations have been run and you have the `users` table in your database.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-confirm-users-table-exists@2x.png' style='max-width:1068px; width:100%' alt=''>



## Controllers
Next, open `/app/Http/Controllers/Auth/` and note the various existing controllers that will be used as part of your user system.

+ `ForgotPasswordController.php`
+ `LoginController.php`
+ `RegisterController.php`
+ `ResetPasswordController.php`


## Routes
Run the command `php artisan make:auth` to generate the routes you'll need for authentication. One outcome of this command will be two new lines at the bottom of `routes/web.php`; the first line is this:

```php
Route::get('/home', 'HomeController@index');
```

By default, the `/home` route is where a user is redirected after they register, login, or logout.

Let's change this route to use `BookController@index` which is the controller/method we've been using as our Foobooks homepage:

```php
Route::get('/home', 'BookController@index');
```

The second line you'll see added to `routes/web.php` is this:
```php
Auth::routes();
```

This single line sets up 10 different routes for your application, including:

+ `/login` (GET & POST)
+ `/logout` (GET)
+ `/password/email` (GET)
+ `/password/reset` (GET & POST)
+ `/password/reset/{token}` (GET)
+ `/register` (GET & POST)

To see all these routes, run `php artisan route:list`. The following screenshot shows the relevant links highlighted:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-auth-routes@2x.png' style='max-width:1180px;' alt=''>




## Views
When we ran ran `php artisan make:auth` to create the auth routes, this command also created the necessary View files we need&mdash; we just need to customize them to match Foobooks.

To begin, open `resources/views/auth/login.blade.php`, which you'll see includes a lot of HTML structure. We want to trim this View down quite a bit, while keeping the essential pieces (email input, password input, &ldquo;remember me&rdquo; checkbox, errors, etc.).

Here's the Foobooks login view after making these edits:
`resources/views/auth/login.blade.php`:

```php
@extends('layouts.master')

@section('content')

    <h1>Login</h1>

    Don't have an account? <a href='/register'>Register here...</a>

    <form id='login' method="POST" action="/login">

        {{ csrf_field() }}

        <label for="email">E-Mail Address</label>
        <input id="email" type="email" name="email" value="{{ old('email') }}" required autofocus>
        @if($errors->has('email'))
            <div class="error">{{ $errors->first('email') }}</div>
        @endif

        <label for="password">Password</label>
        <input id="password" type="password"name="password" required>
        @if ($errors->has('password'))
            <div class="error">{{ $errors->first('email') }}</div>
        @endif

        <label><input type="checkbox" name="remember" {{ old('remember') ? 'checked' : '' }}> Remember Me</label>

        <br>
        <button type="submit" class="btn btn-primary">Login</button>

        <a class="btn btn-link" href="/password/reset">Forgot Your Password?</a>

    </form>

@endsection
```

Similarly, we can trim `resources/views/auth/register.blade.php` down to this:

```php
@extends('layouts.master')

@section('content')

    <h1>Register</h1>

    <form method="POST" id='register' action="{{ route('register') }}">

        {{ csrf_field() }}

        <p>* Required fields</p>

        <label for="name">* Name</label>
        <input id="name" type="text" name="name" value="{{ old('name') }}" required autofocus>
        @if($errors->has('name'))
            <span class="help-block">
                <div class="error">{{ $errors->first('name') }}</div>
            </span>
        @endif

        <label for="email">* E-Mail Address</label>
        <input id="email" type="email" name="email" value="{{ old('email') }}" required>
        @if ($errors->has('email'))
            <span class="help-block">
                <div class="error">{{ $errors->first('email') }}</div>
            </span>
        @endif

        <label for="password">* Password (min: 6)</label>
        <input id="password" type="password" name="password" required>
        @if ($errors->has('password'))
            <div class="error">{{ $errors->first('password') }}</div>
        @endif

        <label for="password-confirm">* Confirm Password</label>
        <input id="password-confirm" type="password" name="password_confirmation" required>

        <br>
        <button type="submit" class="btn btn-primary">Register</button>

@endsection
```




## Test it: Registration
At this point the following components should be set up:

1. The `users` table
2. The `User` Model
3. Registration, login, and log out routes
4. Views for registering and logging in

<br>
Given that, we can test things out. When you visit `/register` you should see a registration form (style may vary):

<img src='http://making-the-internet.s3.amazonaws.com/laravel-register-form@2x.png' style='max-width:343px; width:100%' alt='Register form in Laravel'>

If you fill out this form and submit it, you should be logged in and redirected to `http://localhost/` (assuming validation passed).

How do you know it worked? First, check your `users` table and make sure a new row was added.

Second, you can confirm you are logged in by adding and visiting this temporary route:

```php
Route::get('/show-login-status', function() {

    # You may access the authenticated user via the Auth facade
    $user = Auth::user();

    if($user)
        dump('You are logged in.', $user->toArray());
    else
        dump('You are not logged in.');

    return;
});
```

<img src='http://making-the-internet.s3.amazonaws.com/laravel-confirm-logging-in-worked@2x.png' style='max-width:1018px; width:100%' alt='Confirmed logging in worked'>


## Log out
Revisiting the authorization routes, you'll recall there was a POST route to logout, but no GET route. This makes sense, as the option to log out is typically not delivered via it's own page, but rather presented as a link somewhere in a main heading or menu.

In order to access the logout POST route, we'll need to submit a form, complete with a CSRF token. The trick, however, is we don't want it to look like a form, but instead just a regular link. The following code will accomplish this:

```html
<form method='POST' id='logout' action='/logout'>
    {{ csrf_field() }}
    <a href='#' onClick='document.getElementById("logout").submit();'>Logout</a>
</form>
```

(In the interest of simplicity, this code uses some inline JavaScript to submit the form via a link. Ideally, you should move this inline JavaScript to an external JS file.)

Here's an example of how the logout link could be worked into the Foobooks nav element:
```
<nav>
    <ul>
        <li><a href='/'>Home</a></li>
        <li><a href='/search'>Search</a></li>
        <li><a href='/books/new'>Add a book</a></li>
        <li>
            <form method='POST' id='logout' action='/logout'>
                {{csrf_field()}}
                <a href='#' onClick='document.getElementById("logout").submit();'>Logout</a>
            </form>
        </li>
    </ul>
</nav>
```

<img src='http://making-the-internet.s3.amazonaws.com/laravel-logout-link-added@2x.png' style='max-width:919px;' alt=''>

Test out your new logout link and then and revisit the `/show-login-status` test route to confirm it worked.

(Logically, you won't want the *Logout* link to continue to show when you are no longer logged in; we'll fix that in the next note set.)




## Test it: Log in
We know that logging in automatically occurs when the user registers, now let's test and make sure logging in works independently of registration.

Visit `/login` and fill in the credentials you added during the registration test above to confirm logging in is working.

You can use the `/show-login-status` route again to confirm logging in worked.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-confirm-logging-in-worked@2x.png' style='max-width:648px; width:100%' alt=''>



## Seeds
At this point, you should have a working authentication system complete with login, registration, and logout. In the next note set, we'll look at examples of utilizing authentication, but before we conclude here, let's set up a Seeder for the `users` table.

First step, create the `UsersTableSeeder`:

```bash
$ php artisan make:seeder UsersTableSeeder
```

In the resulting `/database/seeds/UsersTableSeeder.php` file, update the `run` method to add two users:

1. Jill (jill@harvard.edu)
2. Jamal (jamal@harvard.edu).

Both these users will have the same password, `helloworld` (lowercase, no spaces).

```php
public function run()
{

    $user = \App\User::firstOrCreate([
        'email' => 'jill@harvard.edu',
        'name' => 'Jill TestUser',
        'password' => \Hash::make('helloworld')
    ]);

    $user = \App\User::firstOrCreate([
        'email' => 'jamal@harvard.edu',
        'name' => 'Jamal TestUser',
        'password' => \Hash::make('helloworld')
    ]);

}
```

Don't forget to also update `/database/seeds/DatabaseSeeder.php` so the `run` method invokes this new seeder:

```php
$this->call(UsersTableSeeder::class);
```

Test your new seeder to make sure it runs without error.

With this seeding in place, anyone in this course (myself, a TA, a classmate) can log into your site using `jill@harvard.edu` / `helloworld` or `jamal@harvard.edu` / `helloworld`, which will help expedite any troubleshooting assistance you're given.

Obviously, beyond the scope of this academic setting, should you take your application into the &ldquo;real world&rdquo;, you would want to remove these seeds or update them to use a stronger password.




## Reference
+ [Docs: Authentication](http://laravel.com/docs/authentication)
+ [Docs: Hashing](http://laravel.com/docs/hashing)
+ [Docs: Password Resets](http://laravel.com/docs/authentication#resetting-passwords)
