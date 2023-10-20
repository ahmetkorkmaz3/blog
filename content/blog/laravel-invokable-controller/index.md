---
title: Laravel Invokable Controller
date: "2021-09-21T22:12:03.284Z"
description: "laravel, php"
---

Hello to everyone! In this article, we will examine how the `__invoke` method is used in the Laravel controller.

First of all, let's see what the `__invoke` method in php does:
The `__invoke` method is called [**Magic Method**](https://www.php.net/manual/en/language.oop5.magic.php). Used to make a class run as a function.

To use this in Laravel, it is necessary to use `--invokable` as a parameter when creating the controller with the help of `artisan`.

```bash
php artisan make:controller UploadController --invokable
```

After running the bash command above, a controller class will be created as follows:

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\User;

class UploadController extends Controller
{
    public function __invoke()
    {
        // ...
    }
}
```

After doing the work you want to do for the application in the `__invoke` method in the controller, we need to define the controller in `routes`. To define the API, it can be added as follows.

```php
// routes/api.php

Route::post('upload', UploadController::class);
```

As a result, you can use an `invokable` controller when you only want to use a single method in a controller class and you don't need basic methods (index, store, update etc.) in the controller.

Happy coding! :partying_face:
