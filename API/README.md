# API
This section focuses on how to create web services using Laravel.

## Basics
When creating APIs for your project, always create all the API related controller files inside `app\Http\Controllers\API` folder. You don't need to mention the `API` namespace inside `api.php` routes files, rather open up the `App\Providers\RoutesServiceProvider.php` file and concatinate `\API` to the parameter of the `namespace()` function inside the `mapApiRoutes()` method.
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
## Authentication
For authentication, always use `laravel/passport` to provide token based authentication. For details on how to install and configure `passport` for your project, visit the [official documentation](https://laravel.com/docs/5.6/passport#installation) on the Laravel website.
### Custom Validation
By default, passport uses `username` and `password` to perform authentication. If you want to perform custom validation like checking if the user should have `customer` as the type, create a new controller `app\Http\Controllers\API\Auth\TokenController`. Inside the `TokenController` place the following code:
```php
use Psr\Http\Message\ServerRequestInterface;
use Laravel\Passport\Http\Controllers\AccessTokenController;

class TokenController extends AccessTokenController
{
    /**
     * Hooks in before the AccessTokenController issues a token
     *
     *
     * @param  ServerRequestInterface $request
     * @return mixed
     */
    public function issueUserToken(ServerRequestInterface $request)
    {
        $httpRequest = request();

        if ($httpRequest->grant_type == 'password') {

            /**
             * Validating user based on its type.
             * You can place your own logic here.
             */
            $user = \App\Models\User::where([
                ['email', $httpRequest->username],
                ['type', 'customer']
            ])->first();

            return password_verify($httpRequest->password, $user->password) ?
                
                // Issue token if validation succeeds.
                $this->issueToken($request) : 
                
                // Return 422 response if validation fails.
                response()->json([
                    "error" => "invalid_credentials",
                    "message" => "The user credentials were incorrect."
                ], 422);
        }
    }
}
```
Now create a route in `api.php` that routes the request to our custom validation controller.
```php
Route::post('/oauth/token', 'Auth\TokenController@issueUserToken');
```