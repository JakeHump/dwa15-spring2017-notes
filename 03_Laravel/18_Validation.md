# Validation
[Earlier this semester](https://github.com/susanBuck/dwa15-spring2017-notes/blob/master/02_PHP/13_Validation.md), we learned why form validation is important, the kinds of things you should be validating for, and how to do some simple validation with a provided [Form.php](https://github.com/susanBuck/dwa15-php-practice/blob/master/Form.php) class.

Now that we're working in a framework, we want to explore the tools Laravel provides to handle validation.


## Demonstration
As a demonstration, we'll add validation to our *New Book* feature.

To recap, our form for adding a new book looks like this:
```html
<h1>Add a new book</h1>

<form method='POST' action='/books/new'>
    {{ csrf_field() }}

    <label for='title'>Title</label>
    <input type='text' name='title' id='title'>
    <input type='submit' value='Add book'>
</form>
```

That form submits via POST to this route:
```php
Route::post('/books/new', 'BookController@storeNewBook');
```

Which uses this method in `BookController.php`:
```php
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



## Validating the request
Below is a basic validation example where we validate that the incoming `title` for a new book is a) not blank and b) at least three characters long:

```php
public function storeNewBook(Request $request) {

    # Validate the request data
    $this->validate($request, [
        'title' => 'required|min:3',
    ]);

    # Note: If validation fails, it will redirect the visitor back to the form page
    # and none of the code that follows will execute.

    $title = $request->input('title');

    # 
    #
    # [...Code will eventually go here to actually save this book to a database...]
    #
    #

    # Redirect the user to the page to view the book
    return redirect('/books/'.$title);
}
```

Some notes about the above:

1. When using validation, be sure the `Request $request` object is passed to the method. The validator is run on this object, so it's necessary.
2. `$this->validate` is automatically available to your controller methods. You pass it the request object and an array of fields and validation rules.

To test your validation:

1. Submit the form adding a title greater than 3 characters. It should submit as expected.
2. Submit the form *without* adding a title, or adding a title that's less than 3 characters long. It won't show you your confirmation message, but will instead just take your straight back to the form.

At this point validation is working in that it's not completing the `storeNewBook` method if the data is invalid. However, it's not very useful to the user since there's no indication that there was a problem. Your next step is to access and display some error feedback...




## Displaying errors
If validation fails, Laravel automatically redirects the user back to their previous location. In this process, Laravel also makes error messages available to your view in a variable called `$errors` which is an instance of `Illuminate\Support\MessageBag`.

Given this, you can display errors in your view like so:
```html
@if(count($errors) > 0)
    <ul>
        @foreach ($errors->all() as $error)
            <li>{{ $error }}</li>
        @endforeach
    </ul>
@endif

<h1>Add a new book</h1>

<form method='POST' action='/books/new'>
    {{ csrf_field() }}

    <label for='title'>Title</label>
    <input type='text' name='title' id='title'>
    <input type='submit' value='Add book'>
</form>
```

In the above example, we're using the `all` method on `$errors`, one of many methods available when dealing with the error MessageBag.

Alternatively, if we wanted *just* the errors related to the title field (assuming we had more than just the title field present), you could write:

```html
@if($errors->get('title'))
    <ul>
        @foreach($errors->get('title') as $error)
            <li>{{ $error }}</li>
        @endforeach
    </ul>
@endif
```

Read the section on [Working with Error Messages](https://laravel.com/docs/validation#working-with-error-messages) to see more about the methods available to you in the `$error` MessageBag.


## Available validation rules
In the above example, we only saw two rules: `required` and `min`. There are currently 42 total validation rules provided for you from Laravel&mdash; you can read about them here: [Validation: Available Validation Rules](http://laravel.com/docs/validation#available-validation-rules).

## old()



## Required Readings
+ [Laravel Docs: Validation](https://laravel.com/docs/validation) (Skim everything; we're not using all the features provided via Laravel Validation, but it's good to know what's available)
