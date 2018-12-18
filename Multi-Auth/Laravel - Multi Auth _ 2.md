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
-     
