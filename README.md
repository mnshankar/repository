# Caffeinated Repository
**This is a fork of the Caffeinated Repository package for LARAVEL 5.0**

## Getting Started

### Introduction
The Caffeinated Repository package allows the means to implement a standard boilerplate repository interface. This covers the standard Eloquent methods in a non-static, non-facade driven way right out of the box. Fear not though Batman! The Caffeinated Repository package does not limit you in any way when it comes to customizing (e.g overriding) the provided interface or adding your own methods.

If you want database result caching to work, please note that you need a caching solution that supports "tags" (ex. Redis/Memcached).

## Installing Caffeinated Repository
It is recommended that you install the package using Composer.

```
composer require mnshankar/repository
```

This package is compliant with [PSR-1](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md), [PSR-2](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md), and [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md). If you find any compliance oversights, please send a patch via pull request.

# Using Repositories

### Create a Model
Create your model like you normally would. We'll be wrapping our repository around our model to access and query the database for the information we need.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model

class Book extends Model
{
    //
}
```

### Create a Repository
Create a new Repository class - usually these classes are simply stored within a `Repositories` directory. There are a few requirements for each repository instance:

- Repository classes must extend the Caffeinated EloquentRepository class.
- Repository classes must specify a public property pointing to the model.
- Repository classes must specify an array of cache tags. These tags are used by the package to handle automatic cache busting when relevent values change within the database.

```php
<?php

namespace App\Repositories;

use App\Models\Book;
use Caffeinated\Repository\Repositories\EloquentRepository;

class BookRepository extends EloquentRepository
{
    /**
     * @var Model
     */
    public $model = Book::class;

    /**
     * @var array
     */
    public $tag = ['book'];
}
```

### Injecting a Repository
Once you've built and configured your repository instance, you may inject the class within your classes where needed:

```php
<?php

namespace App\Http\Controllers;

use App\Repositories\BookRepository;

class BookController extends Controller
{
    /**
     * @var BookRepository
     */
    protected $book;

    /**
     * Create a new BookController instance.
     *
     * @param  BookRepository  $book
     */
    public function __construct(BookRepository $book)
    {
        $this->book = $book;
    }

    /**
     * Display a listing of all books.
     *
     * @return Response
     */
    public function index()
    {
        $books = $this->book->findAll();

        return view('books.index', compact('books'));
    }
}
```
