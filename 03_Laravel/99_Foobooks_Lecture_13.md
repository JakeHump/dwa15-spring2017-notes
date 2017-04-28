The following is a very rough outline of the modifications I'll make to foobooks during Lecture 13.

This should not be considered a stand-alone document; for full details please refer to the lecture video and the foobooks code source.


## Author dropdown
To associate Authors with Books we'll use a dropdown filled with authors.

To make this happen, we first need an __array of authors__ where the key is the author `id` and the value is the author `name`.

```php
# BookController.php
public function edit($id) {

    $book = Book::find($id);

    # Get all the authors
    $authors = Author::orderBy('last_name', 'ASC')->get();

    # Organize the authors into an array where the key = author id and value = author name
    $authorsForDropdown = [];
    foreach($authors as $author) {
        $authorsForDropdown[$author->id] = $author->last_name.', '.$author->first_name;
    }

    [...]

    # Make sure $authorsForDropdown is made available the view
    return view('book.edit')->with([
        'book' => $book,
        'authorsForDropdown' => $authorsForDropdown
    ]);
```

Then, we can construct the dropdown (`<select>`) using this data:

```html
<label for='author_id'>* Author:</label>
<select id='author_id' name='author_id'>
    @foreach($authorsForDropdown as $author_id => $authorName)
         <option value='{{ $author_id }}' {{ ($book->author_id == $author_id) ? 'SELECTED' : '' }}>
             {{$authorName}}
         </option>
     @endforeach
</select>
```


Now in `saveEdits()` set the `author_id` using the data from the dropdown in `$request`:
```php
$book->author_id = $request->author_id;
```

Test it out to make sure it's working.



## Custom Model Methods
We'll want to do the same thing on the *Add a Book* page.

Rather than duplicate the &ldquo;get authors for dropdown&rdquo; code, we should extract it out of the controller and add it as a method to the `Author` model:

```php
# app/Author.php
public static function authorsForDropdown() {

    $authors = Author::orderBy('last_name', 'ASC')->get();
    $authorsForDropdown = [];
    foreach($authors as $author) {
        $authorsForDropdown[$author->id] = $author->last_name.', '.$author->first_name;
    }

    return $authorsForDropdown;
}
```

Then in `edit()`:
```
$authorsForDropdown = Author::authorsForDropdown();
```

Other frequently used queries can also be abstracted, for example the following method could be added to the Book model:

```php
public static function getAllBooksWithAuthors() {
    return Book::with('author')->orderBy('id','desc')->get();
}
```

## On your own
Update the *Add a Book* page to also have an author dropdown.


## Using Tags/Many to Many
We have everything set up for a tags feature&mdash; migrations, seeders, models&mdash; now let's look at how we'd implement tags.

We need a way to associate tags with books (either from the *Edit Book* or *Create Book* page)

For authors, this was done with a dropdown which worked because each book can have only *one* author.

Each book can have *many* tags, though, so a dropdown won't do. Instead, lets show all possible tags with checkboxes.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-foobooks-tag-checkboxes@2x.png' style='max-width:357px; width:100%' alt='Tags checkboxes'>

To accomplish this, we'll need to gather the following data:

1. All the possible tags
2. All the tags associated with the book we're looking at.


First, a `getTagsForCheckboxes()` method in the Tag model:

```php
# app/Tag.php
public static function getTagsForCheckboxes() {

    $tags = Tag::orderBy('name','ASC')->get();

    $tagsForCheckboxes = [];

    foreach($tags as $tag) {
        $tagsForCheckboxes[$tag['id']] = $tag->name;
    }

    return $tagsForCheckboxes;

}
```

Then update the `edit` method in `BookController`:

```php
# BookController.php
public function edit($id = null) {

    # Get this book and eager load its tags
    $book = Book::with('tags')->find($id);

    # Get authors
    $authorsForDropdown = Author::authorsForDropdown();

    # Get all the possible tags so we can include them with checkboxes in the view
    $tagsForCheckboxes = Tag::getTagsForCheckboxes();

    # Create a simple array of just the tag names for tags associated with this book;
    # will be used in the view to decide which tags should be checked off
    $tagsForThisBook = [];
    foreach($book->tags as $tag) {
        $tagsForThisBook[] = $tag->name;
    }
    # Results in an array like this: $tagsForThisBook => ['novel','fiction','classic'];

    return view('book.edit')
        ->with([
            'book' => $book,
            'authorsForDropdown' => $tagsForThisBook,
            'tagsForCheckbox' => $tagsForCheckboxes,
            'tagsForThisBook' => $tagsForThisBook,
        ]);

}
```

```php
# /resources/views/book/edit.blade.php

[...]

@foreach($tagsForCheckboxes as $id => $name)
    <input
        type='checkbox'
        value='{{ $id }}'
        name='tags[]'
        {{ (in_array($name, $tagsForThisBook)) ? 'CHECKED' : '' }}
    >
    {{ $name }} <br>
@endforeach

[...]
```


```php
# BookController.php
public function update(Request $request, $id) {

    # [...Validation removed for brevity...]

    # Find and update book
    $book = Book::find($request->id);
    $book->title = $request->title;
    $book->cover = $request->cover;
    $book->published = $request->published;
    $book->purchase_link = $request->purchase_link;
    $book->save();

    # If there were tags selected...
    if($request->tags) {
        $tags = $request->tags;
    }
    # If there were no tags selected (i.e. no tags in the request)
    # default to an empty array of tags
    else {
        $tags = [];
    }

    # Above if/else could be condensed down to this: $tags = ($request->tags) ?: [];

    # Sync tags
    $book->tags()->sync($tags);
    $book->save();

    # [... Finish removed for brevity ..]

}
```

Test it out by removing and adding tags.


## Revisiting the book delete feature

Now that books are associated with tags, you'll get a *foreign key constraint* SQL error when trying to delete a book. To fix, you want to remove any tag associations before the book is deleted...

```php
public function delete(Request $request) {

    # Get the book to be deleted
    $book = Book::find($request->id);

    if(!$book) {
        Session::flash('message', 'Deletion failed; book not found.');
        return redirect('/books');
    }

    # Remove any tag associations with this book
    $book->tags()->detach();

    $book->delete();

    # [...etc...]

}
```

