# Laravel Wiki
Tips, tricks and how-tos for your Laravel projects.

## Authentication
Use `php artisan make:auth` command to implement authentication in your Laravel project. Visit the [offical documentation](https://laravel.com/docs/5.6/authentication) for more details on how to setup authentication.
### Custom Validation
There are two different ways to implement custom validation of the user credentials, depending on the scenario.
If you want to check if a user being authenticated has a `role` of `admin`, then you just need to override the `credentials()` method of the `AuthenticatesUsers` trait being used inside the `LoginController`.
```php
/**
 * Get the needed authorization credentials from the request.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
protected function credentials(Request $request)
{
    return [
        $this->username() => $request->{$this->username()},
        'password' => $request->password,
        'role' => 'admin'
    ];
}
```
If you want to check if a user being authenticated has a `role` other than `customer`, then you need to override the `attemptLogin()` method of the `AuthenticatesUsers` trait being used inside the `LoginController`.
```php
/**
 * Attempt to log the user into the application.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return bool
 */
protected function attemptLogin(Request $request)
{
    if (
        $this->guard()->attempt(
            $this->credentials($request), $request->filled('remember')
        ) && $this->guard()->user()->type != 'customer'
    ) return true;
    
    $this->guard()->logout();
    return false;
}
```