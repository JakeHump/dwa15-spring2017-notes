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


## Before you go: seed data
In the next note set we'll explore examples of querying the `books` table in the `foobooks` database, so it'll be useful if we're all working with the same data.

On your local server, in phpMyAdmin, select the `foobooks` database then find the *SQL* tab.

Copy/paste and run the following SQL:

```sql
DELETE FROM books;
INSERT INTO `books` (`id`, `created_at`, `updated_at`, `title`, `author`, `published`, `cover`, `purchase_link`) VALUES
(1, '2017-04-13 13:19:36', '2017-04-13 13:19:36', 'The Great Gatsby', 'F. Scott Fitzgerald', 1925, 'http://img2.imagesbn.com/p/9780743273565_p0_v4_s114x166.JPG', 'http://www.barnesandnoble.com/w/the-great-gatsby-francis-scott-fitzgerald/1116668135?ean=9780743273565'),
(2, '2017-04-13 13:19:36', '2017-04-13 13:19:36', 'The Bell Jar', 'Sylvia Plath', 1963, 'http://img1.imagesbn.com/p/9780061148514_p0_v2_s114x166.JPG', 'http://www.barnesandnoble.com/w/bell-jar-sylvia-plath/1100550703?ean=9780061148514'),
(3, '2017-04-13 13:19:36', '2017-04-13 13:19:36', 'I Know Why the Caged Bird Sings', 'Maya Angelou', 1969, 'http://img1.imagesbn.com/p/9780345514400_p0_v1_s114x166.JPG', 'http://www.barnesandnoble.com/w/i-know-why-the-caged-bird-sings-maya-angelou/1100392955?ean=9780345514400'),
(4, '2017-04-13 13:19:36', '2017-04-13 13:19:36', 'Harry Potter and the Sorcerer\'s Stone', 'J.K. Rowling', 1997, 'http://prodimage.images-bn.com/pimages/9780590353427_p0_v1_s484x700.jpg', 'http://www.barnesandnoble.com/w/harry-potter-and-the-sorcerers-stone-j-k-rowling/1100036321?ean=9780590353427'),
(5, '2017-04-13 13:19:36', '2017-04-13 13:19:36', 'Harry Potter and the Chamber of Secrets', 'J.K. Rowling', 1998, 'http://prodimage.images-bn.com/pimages/9780439064873_p0_v1_s192x300.jpg', 'http://www.barnesandnoble.com/w/harry-potter-and-the-chamber-of-secrets-j-k-rowling/1004338523?ean=9780439064873'),
(6, '2017-04-13 13:19:36', '2017-04-13 13:19:36', 'Harry Potter and the The Prisoner of Azkaban', 'J.K. Rowling', 1999, 'http://prodimage.images-bn.com/pimages/9780439136365_p0_v1_s192x300.jpg', 'http://www.barnesandnoble.com/w/harry-potter-and-the-prisoner-of-azkaban-j-k-rowling/1100178339?ean=9780439136365');


```

Example:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-foobooks-sql-seed@2x.png' style='max-width:1343px;' alt=''>

After you run the query, your `books` table should have three rows:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-foobooks-sql-seed-results@2x.png' style='max-width:1126px;' alt=''>

With some data to work with, proceed to the next note set.


