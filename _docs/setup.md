## Installation

You can install the package via composer:

```bash
composer require mexion/multilingual-admin
```

You can publish and run the migrations with:

```bash
php artisan vendor:publish --provider="Mexion\MultilingualAdmin\MultilingualAdminServiceProvider" --tag="migrations"
php artisan migrate
```

You can publish the config file with:
```bash
php artisan vendor:publish --provider="Mexion\MultilingualAdmin\MultilingualAdminServiceProvider" --tag="config"
```

## Setting up middelware
Add the following middleware to your routes web so that the cookies are captured
```
'web' => [
    ...
    \Mexion\MultilingualAdmin\Http\Middleware\ReferrerMiddleware::class,
```

## Without publishing your views and you use of VUE:
In your app.js
```
require("./../../vendor/mexion/multilingual-admin/resources/assets/package.js")
```
rerun
```
run npm prod
```

# Views publishing
### Blade version
```bash
php artisan vendor:publish --provider="Mexion\MultilingualAdmin\MultilingualAdminServiceProvider" --tag="blade"
```

### Vue version
Note , this needs vuetify v-datatable and v-chip
```bash
php artisan vendor:publish --provider="Mexion\MultilingualAdmin\MultilingualAdminServiceProvider" --tag="vue"
```

in your app.js
```
require("./../assets/vendor/multilingual-admin/package")
```

rerun
```
run npm prod
```


# Register your routes
### User routes
For referrers to see their own statistics
Make sure you publish these routes within your authentication middleware
```
Route::MultilingualAdminUser('multilingual-admin');
```

This makes the routes available as
/multilingual-admin/details

### Admin routes
Your admin routes to see the referrers overview
Make sure you publish these routes within your authentication middleware
```
Route::MultilingualAdminAdmin('multilingual-admin');
```

This makes the routes available as
/multilingual-admin/referrers

## Usage

``` php
$multilingual-admin = new Mexion\MultilingualAdmin();
echo $multilingual-admin->echoPhrase('Hello, Mexion!');
```

## Testing

``` bash
composer test
```

## Testing frontend
First update the packages by running npm run prod. This will update the frontend in the orchastra package
then run the actual tests
``` bash
npm run prod
composer test-browser
```
