# Laravel - Multi Auth : 2
- Now when you open **http://app-name.test/admin** you can see the login page of users instead of admins
- for solving this, open **LoginController.php** and find:
```php
use AuthenticatesUsers;
```
- open this file, i.e. **AuthenticateUsers.php** in **vendor/laravel/framework/src/Illuminate/Foundation/Auth** folder
- copy the **showLoginForm()** method and paste into **LoginController.php**
```php
/**
     * Show the application's login form.
     *
     * @return \Illuminate\Http\Response
     */
    public function showLoginForm()
    {
        return view('auth.login');
    }
```
-     update as follows, here we are overwriting the **showLoginForm()** method in **AuthenticateUser.php** for the **admins**
```php
/**
     * Show the application's login form.
     *
     * @return \Illuminate\Http\Response
     */
    public function showLoginForm()
    {
        return view('admin.login');
    }
```
- next open **AdminController.php** and uncomment the **middleware** in **constructor**
```php
public function __construct()    
    {
        $this->middleware('auth:admin');
    }
```
- next copy the **guard()** method from the **AuthenticateUsers.php**  and overwrite in the **LoginController.php** in Admin
```php

use Illuminate\Support\Facades\Auth;
/**
     * Get the guard to be used during authentication.
     *
     * @return \Illuminate\Contracts\Auth\StatefulGuard
     */
    protected function guard()
    {
        return Auth::guard('admin');
    }
```
- Now you can login into the admin and see the admin dashboard
---
- Now when we logged in as User and try to access admin dashboard, I.e **admin/home** we will redirected to the Users dashboard.
- For redirect the users to Admin Login page when try to access the **admin/home,** go to **RedirectIfAuthenticated.php middleware** and update **handle()** method as follows
```php
public function handle($request, Closure $next, $guard = null)
    {   
        switch ($guard) {

            case 'admin':
            
                if (Auth::guard($guard)->check()) {
            
                    return redirect('admin/home');
            
                }
            
                break;
            
            default:
                
                if (Auth::guard($guard)->check()) {
                
                    return redirect('/home');
                
                }
                
                break;

        }

        return $next($request);
    
    }
```
- Now open the **Handler.php** in **Exceptions** folder and paste the following code:
```php
/**
     * Convert an authentication exception into an unauthenticated response.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Illuminate\Auth\AuthenticationException  $exception
     * @return \Illuminate\Http\Response
     */
    protected function unauthenticated($request, AuthenticationException $exception)
    {
        if ($request->expectsJson()) {
            return response()->json(['error' => 'Unauthenticated.'], 401);
        }

        $guard = array_get($exception->guards(),0);

        switch ($guard) {

            case 'admin':

                return redirect()->guest(route('admin.login'));

                break;
            
            default:

                return redirect()->guest(route('login'));

                break;

        }
```
- Now when we try to access the **admin/home** we will be redirected into admin login page.
- Finished!
