# API
This section focuses on how to create web services using Laravel.

### Basics
When creating APIs for your project, always create all the API related controller files inside `app\Http\Controllers\API` folder. You don't need to mention the `API` namespace inside `api.php` routes files, rather open up the `App\Providers\RoutesServiceProvider.php` file and concatinate the `\API` to the parameter of the `namespace()` function inside the `mapApiRoutes()` method.
```php
/**
 * Define the "api" routes for the application.
 *
 * These routes are typically stateless.
 *
 * @return void
 */
protected function mapApiRoutes()
{
    Route::prefix('api')
         ->middleware('api')
         ->namespace($this->namespace . '\API')
         ->group(base_path('routes/api.php'));
}
```