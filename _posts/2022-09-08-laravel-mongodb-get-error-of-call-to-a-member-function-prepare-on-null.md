---
layout: post
title: Laravel mongodb get error of Call to a member function prepare() on null
date: 2022-09-08 18:20 +0530
categories: [Tech, Laravel, PHP, MongoDB]
tags: [tech, laravel, php, mongodb]
---

![Laravel Socialite](/assets/img/posts/laravel-mongodb.jpg){: w="700" h="250" }

So long time ago I decided to use mongodb with laravel.

I used this package from [laravel-mongodb](https://github.com/jenssegers/laravel-mongodb) package from [@jenssegers](https://github.com/jenssegers), it was very easy to install mongodb package into laravel.

> ```composer require jenssegers/mongodb```

After adding database credentials in ```.env``` I tried running database migrations, surprisingly the laravel migration were also ran without any errors.

This issue occurred when I tried inserting data in User collection.

> ```php laravel get error of Call to a member function prepare() on null```


The issue was that laravel User Model by default uses this class ```Illuminate\Foundation\Auth\User``` as ```Authenticatable``` which failed when using mongodb.

The solution was to use ```Jenssegers\Mongodb\Auth\User as Authenticatable``` in place of ```Illuminate\Foundation\Auth\User as Authenticatable```

Here is how my model looked after the change:

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
//use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;
use Jenssegers\Mongodb\Auth\User as Authenticatable; // Added this

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;
    
    ````
}
```

This solved the problem for me, I hope it helps others also.