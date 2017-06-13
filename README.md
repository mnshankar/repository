# Caffeinated Repository
**This is a fork of the Caffeinated Repository package for LARAVEL 5.0 (using PHP 5.4)**

## Getting Started

### Introduction
The Caffeinated Repository package implements a standard boilerplate repository interface. This covers the standard Eloquent methods in a non-static, non-facade driven way right out of the box. Fear not though Batman! The Caffeinated Repository package does not limit you in any way when it comes to customizing (e.g overriding) the provided interface or adding your own methods.

If you want database result caching to work, please note that you need a caching solution that supports "tags" (ex. Redis/Memcached). This package automatically caches data retrieved through the repository (for 60 mins). It also handles cache busting (based on tags) for data that is updated via the repository. In fact, it is for this feature that Caffeinated/repository requires the use of a caching store with Tags feature - It makes removing chunks of data from the cache easy.

## Installing Caffeinated Repository
It is recommended that you install the package using Composer.

```
composer require mnshankar/repository
```
IMPORTANT: Also add the service provider to your app/config.php file - This sets up eventing so updates through your repository invalidate the cache, thus forcing a requery.
 ```
 'Caffeinated\Repository\RepositoryServiceProvider'
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

- Repository classes must extend the EloquentRepository class.
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
Once you've built and configured your repository instance, you may inject the class within your controller classes where needed:

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
The interface defined in src/Contracts/Repository.php details the methods implemented by the EloquentRepository class. These are listed below for your convenience:
```
public function find($id, $columns = ['*'], $with = []);
public function findBy($attribute, $value, $columns = ['*'], $with = []);
public function findAll($columns = ['*'], $with = []);
public function findWhere($where, $columns = ['*'], $with = []);
public function findWhereBetween($attribute, $values, $columns = ['*'], $with = []);
public function findWhereIn($attribute, $values, $columns = ['*'], $with = []);
public function findWhereNotIn($attribute, $values, $columns = ['*'], $with = []);
public function findOrCreate($attributes);

//RAISES EVENTS 'tag'.entity.creating and 'tag'.entity.created 
public function create($attributes);
//RAISES EVENTS 'tag'.entity.updating and 'tag'.entity.updated
public function update($id, $attributes);
//RAISES EVENTS 'tag'.entity.deleting and 'tag'.entity.deleted
public function delete($id);

public function with($relationships);
public function orderBy($column, $direction = 'asc');

//BE CAREFUL WHEN YOU USE PLUCK.. SEMANTICS HAVE CHANGED BETWEEN Laravel 5.0 AND FUTURE VERSIONS
public function pluck($column, $key = null);

public function paginate($perPage = null, $columns = ['*'], $pageName = 'page', $page = null);
public static function __callStatic($method, $parameters);
public function __call($method, $parameters);
```

Event listeners in "Listeners\RepositoryEventListener" take care of invalidating cache on create, update and delete.
Note that you may manually flush the cache associated with any of your repositories using
```
$repositoryObject->flushCache();
```

While these methods should take care of 80% of your needs, you may define additional methods as required by your app in your repository class.
Refer to the EloquentRepository class for implementation details.
