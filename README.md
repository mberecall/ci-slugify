<p align="center">

![GitHub release](https://img.shields.io/github/v/release/mberecall/ci4-slugify) <img alt="GitHub code size in bytes" src="https://img.shields.io/github/languages/code-size/mberecall/ci4-slugify"> [![Package License](https://img.shields.io/badge/License-MIT-brightgreen.svg)](LICENSE) [![Buy me a coffee](https://img.shields.io/static/v1.svg?label=Buy%20me%20a%20coffee&message=🥨&color=black&logo=buy%20me%20a%20coffee&logoColor=white&labelColor=6f4e37)](https://www.ko-fi.com/mberecall) [![Packagist Downloads](https://img.shields.io/packagist/dm/mberecall/ci4-slugify)](https://packagist.org/packages/mberecall/ci4-slugify) [![Type Coverage](https://shepherd.dev/github/mberecall/ci4-slugify/coverage.svg)](https://shepherd.dev/github/mberecall/ci4-slugify)

</p>

Contents
-------- 
* [What is a slug?](#background-what-is-a-slug)
* [Installation](#installation)
* [Usage](#usage )
   * [Update modal](#1-update-modal)
   * [The SlugService Class](#2-the-slugservice-class)



# Simple Slugify Library for Codeigniter 4

This library provides a way of generating a unique slugs.

<p align="center">  

 [![Hello World](img/ko-fi.png)](https://ko-fi.com/mberecall)

 </p>

## Background: What is a slug?
A slug is a simplified version of a string, typically URL-friendly. The act of "slugging" 
a string usually involves converting it to one case, and removing any non-URL-friendly 
characters (spaces, accented letters, ampersands, etc.). The resulting string can 
then be used as an identifier for a particular resource.

For example, if you have a blog with posts, you could refer to each post via the ID:

    http://yourdomain.com/post/1
    http://yourdomain.com/post/2

... but that's not particularly friendly (especially for 
[SEO](http://en.wikipedia.org/wiki/Search_engine_optimization)). You probably would 
prefer to use the post's title in the URL, but that becomes a problem if your post 
is titled "André & François took 55% of the whole space in REAC Company", because this is pretty ugly too:

  http://yourdomain.com/post/Andr%C3%A9%20&%20Fran%C3%A7ois%20took%2055%%20of%20the%20whole%20space%20in%20REAC%20Company

The solution is to create a slug from the title and use that instead. You might want 
to use Codeigniter's built-in `\url_title('This is the title','-', true)` helper function to convert that title/string into slug as follow:

    http://yourdomain.com/post/andre-francois-took-55-of-the-whole-space-in-reac-company


A URL like that will make users happier (it's readable, easier to type, etc.).

For more information, you might want to read 
[this](http://en.wikipedia.org/wiki/Slug_(web_publishing)#Slug) description on Wikipedia.

Slugs tend to be unique as well. So if you write another post with the same title, 
you'd want to distinguish between them somehow, typically with an incremental counter 
added to the end of the slug:

    http://yourdomain.com/post/andre-francois-took-55-of-the-whole-space-in-reac-company
    http://yourdomain.com/post/andre-francois-took-55-of-the-whole-space-in-reac-company-1
    http://yourdomain.com/post/andre-francois-took-55-of-the-whole-space-in-reac-company-2

This keeps the URLs unique.

## Installation

This package can be installed through `composer require`. Before install this, make sure that your are working with PHP >= 7.2 in your system.
Just run the following command in your cmd or terminal:

> Install the package via Composer:

```bash
composer require mberecall/ci4-slugify
```



## Usage

After package installed in your Codeigniter project, you have two ways you will use to generate a unique slugs for your blog posts or products. One way, is to use `CI_Slugify` class inside model. This means that you will need to update your model. The second way, you will use SlugService Class inside your controller.

### [1] Update modal

`CI_Slugify` is intended to be used from a Model as it requires one. Here is an example of the default usage:

```php
<?php

namespace App\Models;

use CodeIgniter\Model;
use \Mberecall\Sluggable\CI_Slugify;

class Post extends Model
{
    protected $table = 'posts';
    protected $allowedFields = ['title', 'content'];

    // Callbacks
    protected $beforeInsert = ['setSlug'];
    protected $beforeUpdate = ['setSlug'];

     public function setSlug($data)
     {
        $slugify = new CI_Slugify($this);
        // $slugify->field('slug_field'); // default: `slug`
        $data = $slugify->getSlug($data,'title');
        return $data;
    }
}
```
`NB:` If you use a slug field that is not called `slug`, you can call `$slugify->field('your_slug_field_name')` to change the default behaviour.

This code will look for the `title` value of the incomming data as source and calculate the corresponding `slug`. If a database entry already has that given `slug`, it will increment the slug with a `-N` suffix. For instance: `hello-world` becomes `hello-world-2`, which becomes `hello-world-3`, and so on until a free `slug` is found.

In your controller, no need to add a slug field on the array when you are inserting data into database. See the example below:

```php
<?php

namespace App\Controllers;

use App\Controllers\BaseController;
use App\Models\Post;

class TestController extends BaseController
{
    public function index(){
        $post = new Post();
        $post->save([
            'title'=>'André & François won mathematics competion',
            // 'slug'=>$slug  you don't need to add this on array
        ]);

        //Result: andre-francois-won-mathematics-competion
       //         andre-francois-won-mathematics-competion-1      
    }   
}
```

That's it ... your model is now "sluggable"!

### [2] The SlugService Class 
All the logic to generate slugs is handled
by the `Mberecall\CI_Slugify\SlugService` class.

Generally, you don't need to access this class directly, although there is one 
static method that can be used to generate a slug for a given string without actually
creating or saving an associated model. All you need to do, is to use the below strategies in your controller in order to generate a unique slugs.

```php
<?php

namespace App\Controllers;

use App\Controllers\BaseController;
use App\Models\Post;
use \Mberecall\CI_Slugify\SlugService;

class TestController extends BaseController
{
    public function index(){
        $post = new Post();
        $title = 'André & François won mathematics competion';
        $slug = SlugService::table('posts')->make($title); //When you use table name
        $slug = SlugService::model(Post::class)->make($title); //When you use model object
        //Result for $slug: andre-francois-won-mathematics-competion-3
        $post->save([
            'title'=>$title,
            'slug'=>$slug 
        ]);
    }   
}
```

Below are possible examples you can use to generate unique slug in your controller using SlugService class:

```php
<?php

namespace App\Controllers;

use App\Controllers\BaseController;
use App\Models\Post;
use \Mberecall\CI_Slugify\SlugService;

class TestController extends BaseController
{
    public function index(){

        $post = new Post();
        $title = 'André & François won mathematics competion';

        /** Default way. Both lines will return same result */
        $slug = SlugService::table('posts')->make($title);
        $slug = SlugService::model(Post::class)->make($title);

        /** When you specify slug field name. defaul is 'slug' */
        $slug = SlugService::table('posts')->make($title,'post_slug');
        $slug = SlugService::model(Post::class)->make($title,'post_slug');

        /** When you specify divider/separator for generated slug. default is '-' */
        $slug = SlugService::table('posts','id')->make($title);
        $slug = SlugService::model(Post::class,'id')->make($title);
    
       /** When you specify divider/separator for generated slug. default is '-' */
        $slug = SlugService::table('posts')->separator('_')->make($title);
        $slug = SlugService::model(Post::class)->separator('_')->make($title);
            
        //Result for $slug: 1: andre-francois-won-mathematics-competion
        //Result for $slug: 2: andre_francois_won_mathematics_competion   
    }   
}

```

## Class reference

### `\Mberecall\CI_Slugify\SlugService`

#### `table()` 

**Arguments**
- This method has two arguments. First argument will be the name of table which is required. Where the second argument is the primary key of the modal table. eg: `model('products','pid')`. Default value of the second argument which is primary key is 'id'. This means that we suppose that the products table has id column/field in structure.

#### `model()` 

**Arguments**
- This method has two arguments. First argument will be the instance of the model object which is required. Where the second argument is the primary key of the modal table. eg: `model(Product::class,'pid')`. Default value of the second argument which is primary key is 'id'. This means that we suppose that the products table has id column/field in structure.

#### `separator()` 

**Arguments**
- This method has one argument. Here, you can specify the devider that will be included in generated slug. eg: `separator('_')`. The default value is dash `'-'`.

#### `make()` 

**Arguments**
- This method has two arguments. First argument is the string value from title which is required. Second argument is the slug field name. You can specify the name of field name in this method. The default value of second argument is `'slug'`. eg: `make('Title Of Your Choice','slug')`

## Copyright and License

[ci4-slugify](https://github.com/mberecall/ci4-slugify)
was written by [MB'DUSENGE Callixte (mberecall)](https://github.com/mberecall) and is released under the 
[MIT License](LICENSE.md).

Copyright (c) 2023 MB'DUSENGE Callixte