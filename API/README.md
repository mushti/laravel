# API
This section focuses on how to create web services using Laravel.
1. [Basics](#basics)
2. [Authentication](#authentication)
    * [Custom Validation](#custom-validation)
        1. [Using `User` Model](#i-using-user-model)
        2. [Using `AccessTokenController`](#ii-using-accesstokencontroller)
## Basics
[[Back to Index](#api)] When creating APIs for your project, always create all the API related controller files inside `app\Http\Controllers\API` folder. You don't need to mention the `API` namespace inside `api.php` routes files, rather open up the `App\Providers\RoutesServiceProvider.php` file and concatinate `\API` to the parameter of the `namespace()` function inside the `mapApiRoutes()` method.
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
[[Back to Index](#api)] For authentication, always use `laravel/passport` to provide token based authentication. For details on how to install and configure `passport` for your project, visit the [official documentation](https://laravel.com/docs/5.6/passport#installation) on the Laravel website.
### Custom Validation
By default, passport uses `username` and `password` to perform authentication. If you want to perform custom validation like checking if the user should have `customer` as the role, there are two different methods to achieve this logic.
##### i. Using `User` Model
[[Back to Index](#api)] In the `User` class where you have used `HasApiToken` trait, you can define `findForPassport()` method in which you can define your custom validation logic.
```php
/**
 * Find user for passport.
 *
 * @param  string $username
 * @return mixed
 */
public function findForPassport($username) {
    return $this->where('email', $username)->where('role', 'customer')->first();
}
```
Similarly, if you want to play with the `password` field, you can define `validateForPassportPasswordGrant()` method in the `User` class to override default logic. Like in the following example I have completely removed the password validation check by returning `true`.
```php
/**
 * Override password validation logic for passport.
 *
 * @param  string $password
 * @return boolean
 */
public function validateForPassportPasswordGrant($password) {
    return true;
}
```
##### ii. Using `AccessTokenController`
[[Back to Index](#api)] Another method to achieve custom validation, create a new controller called the `TokenController` which extends the `AccessTokenController` class or, if you already have an `AccountController` you can also extend it with `AccessTokenController`. Inside the `TokenController` or `AccountController`, place the following code:
```php
use Psr\Http\Message\ServerRequestInterface;
use Laravel\Passport\Http\Controllers\AccessTokenController;

class TokenController extends AccessTokenController
{
    /**
     * Hooks in before the AccessTokenController issues a token
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

            return ($user && password_verify($httpRequest->password, $user->password)) ?
                
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
> If you only want the user to authenticate with your custom validation controller, don't forget to remove the `Passport::routes()` method from the `boot()` method of your `app\Providers\AuthServiceProvider.php` file.
> But, doing this will remove all other `passport` routes registered for other features available in the library.