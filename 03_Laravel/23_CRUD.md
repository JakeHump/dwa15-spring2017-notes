# CRUD: Create, Read, Update, Delete

All database interactions can be boiled down to these four actions:

1. Create - add new rows to a table
2. Read - query for existing rows in a table
3. Update - change rows in a table
4. Delete - delete row(s) in a table

These actions are often abbreviated using the acronym **CRUD (Create, Read, Update, Delete)**.

## CRUD Examples
Imagine a generic e-commerce store:

1. When you place an order, a new row is added to an `orders` table. (Create)

2. When you go to an order history page, the `orders` table is queried for any order associated with your shopper id. (Read)

3. When your order is shipped, the corresponding row in the `orders` table is updated so that a field called `status` is changed from `pending` to `shipped`. (Update)

4. When you go to your account settings and delete a saved shipping address, the corresponding row in an `addresses` table is removed. (Delete)


## Laravel -> Database
To interact with the database (and *CRUD* data), Laravel has two tools:

__The first tool is the [Query Builder](https://laravel.com/docs/5.4/queries), accessed via the DB facade:__

>> *&ldquo;Laravel's database query builder provides a convenient, fluent interface to creating and running database queries. It can be used to perform most database operations in your application and works on all supported database systems. &rdquo;*


Query Builder example statements:
```php
DB::table('posts')->get(); # Get all the books
DB::table('posts')->where('id',$id)->first(); # Find a book
DB::table('posts')->where('id',$id)->delete(); # Delete a book
```


__The second tool is [Eloquent ORM](https://laravel.com/docs/5.4/eloquent) (Object-relational Mapping):__

>> *&ldquo;The Eloquent ORM included with Laravel provides a beautiful, simple ActiveRecord implementation for working with your database. Each database table has a corresponding &lsquo;Model&rsquo; which is used to interact with that table. Models allow you to query for data in your tables, as well as insert new records into the table.&rdquo;*

Eloquent example statements:

```php
Book::all() # Get all the books
Book::find($id) # Find a book
Book::delete($id) # Delete a book
```


## Which tool to use?
Some projects/developers use only the Query Builder, while some use only Eloquent.

Most projects/developers use some combination of both, with a favoring towards Eloquent.

Neither tool is necessarily better than the other, they're just different with their own pros and cons.

In the interest of time, __we'll focus our attention in this course on using Eloquent ORM__.

(If you have experience with writing code to interact with databases, [this article](https://blog.sriraman.in/laravel-eloquent-vs-fluent-query-builder/) does a good job of explaining the differences and pros/cons of Query Builder vs. Eloquent).


## Summary
+ Our applications will need to Create, Read, Update, and Delete data.
+ To do this, we'll use Eloquent ORM.
