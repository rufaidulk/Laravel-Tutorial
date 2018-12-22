# Laravel - Multi Auth : 4 - User roles (ACL)
> Note: ACL stands for Access Control List
- So first creat the model and migration for roles
```php
php artisan make:model Role -m
php artisan make:model RoleAdmin -m
```
- Update the schema of the **roles** migration as follows:
```php
public function up()
    {
        Schema::create('roles', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->timestamps();
        });
    }
```
- Update the schema of the **role_admins** migration as follows:
```php
public function up()
    {
        Schema::create('role_admins', function (Blueprint $table) {
            $table->increments('id');
            $table->integer('role_id')->unsigned(();
            $table->integer('admin_id')->unsigned(();
            $table->timestamps();
        });
    }
```
-     Now run the **`php artisan migrate`**
---
- Now manually insert a new admin to admin table, i.e via phpMyAdmin
- Also add 2 roles for **role** table 
1. admin // id will be 1
1. editor // id will be 2
- insert data as follows in **role_admins** table
```php
role_id     admin_id
1              1     //means first user in admin table is admin
2              2     //means second user in admin table is editor
```
- Now lets describe the relationships between role and admin, go to **Admin** model and update as follows:
```php
/*======================================================
    |   
    |   relationships
    */

    public function role()
    {

        return $this->belongsToMany(Role::class, 'role_admins');

    }
```
- For checking everything working correct or not, go to **tinker,** run this command:
```php
php artisan tinker
```
- This wiil give you the tinker shell, inside run beloow command:
```php
App\Admin::find(1)
```
- you will get an output like this:
```php
=> App\Admin {#2923
     id: 1,
     name: "name of the admin",
     email: "email id of the admin",
     email_verified_at: null,
     created_at: null,
     updated_at: "2018-12-21 09:01:19",
   }
```
- Now run this command:
```php
App\Admin::find(1)->role
```
- You will get an output like this:
```php
=> Illuminate\Database\Eloquent\Collection {#2928
     all: [
       App\Role {#2927
         id: 1,
         name: "admin",
         created_at: null,
         updated_at: null,
         pivot: Illuminate\Database\Eloquent\Relations\Pivot {#2916
           admin_id: 1,
           role_id: 1,
         },
       },
     ],
   }
```
- I.e relationships are working properly, so **exit** tinker by running exit 
---
- Now we need to  go to **LoginController** in **admin** folder and find this line:
```php
use AuthenticatesUsers;
```
- open this **AuthenticatesUsers.php** in **vendor/laravel/framework/src/Illuminate/foundation/Auth** folder and find the **login()** method
```php
/**
     * Handle a login request to the application.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\RedirectResponse|\Illuminate\Http\Response|\Illuminate\Http\JsonResponse
     *
     * @throws \Illuminate\Validation\ValidationException
     */
    public function login(Request $request)
    {
        $this->validateLogin($request);

        // If the class is using the ThrottlesLogins trait, we can automatically throttle
        // the login attempts for this application. We'll key this by the username and
        // the IP address of the client making these requests into this application.
        if ($this->hasTooManyLoginAttempts($request)) {
            $this->fireLockoutEvent($request);

            return $this->sendLockoutResponse($request);
        }

        if ($this->attemptLogin($request)) {
            return $this->sendLoginResponse($request);
        }

        // If the login attempt was unsuccessful we will increment the number of attempts
        // to login and redirect the user back to the login form. Of course, when this
        // user surpasses their maximum number of attempts they will get locked out.
        $this->incrementLoginAttempts($request);

        return $this->sendFailedLoginResponse($request);
    }
```
-     you can see the **sendLoginResponse()**  which makes us to login and redirect to the home, open this method in the same file
```php
/**
     * Send the response after the user was authenticated.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    protected function sendLoginResponse(Request $request)
    {
        $request->session()->regenerate();

        $this->clearLoginAttempts($request);

        return $this->authenticated($request, $this->guard()->user())
                ?: redirect()->intended($this->redirectPath());
    }
```
- just copy and paste it into the **LoginController** in **Admin** and update as follows:
```php
use Illuminate\Http\Request;
/**
     * Send the response after the user was authenticated.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    protected function sendLoginResponse(Request $request)
    {
        $request->session()->regenerate();

        $this->clearLoginAttempts($request);

        foreach ($this->guard()->user()->role as $role) {
            
            if ($role->name == "admin") {
                
                return redirect('admin/home');

            } elseif ($role->name == "editor") {
                
                return redirect('admin/editor');

            }

        }
```
-     this redirects admin users based on the role to different views. after that go to **web.php** and create the route **admin/editor** 
```php
Route::get('admin/editor', 'EditorController@index');
```
- Now create the **EditorController** by running :
```php
php artisan make:controller EditorController
```
- After creating the controller, let define the **index()** method as follows in the controller
```php
public function index()
    {
    	return view('admin.editor');
    }
```
- Now create the **editor view** mentioned above, for that create a new file **editor.blade.php** in **admin** folder and copy below content to that.
```php
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">Editor Dashboard</div>

                <div class="card-body">
                    @if (session('status'))
                        <div class="alert alert-success" role="alert">
                            {{ session('status') }}
                        </div>
                    @endif

                    You are logged in as Editor!
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```
- Now when you go to the app and login with credentials of the editor you will be redirected into editor dashboard
- but the logout link is not showing in the editor dashboard, lets fix it
- for that open **AdminController** and copy the admin middleware and paste into the **EditorController,** Now refresh the page you can see the logout system in Editor dashboard
```php
/**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth:admin');
    }
```

---
- Now whe you login  as admin and try to access the editor dashboard i.e **admin/editor,** you are able to access and also you can access the admin dashboard after login as an editor.
- To prevent this create a **middleware** by running:
```php
php artisan make:middleware EditorMiddleware
```
- Open the **EditorMiddleware** in **App/Http/Middleware** folder and update the **handl()** method as follows
```php
use Auth;

public function handle($request, Closure $next)

    {
        
        foreach (Auth::user()->role as $role) {
            
            if ($role->name == "editor") {
                
                return $next($request);
            }
        }

        return redirect('/');

    }
```
- Here the middleware check the admin user is editor, if it is alllow to go with request, otherwise it will redirect to welcome page.
- before that register our new middleware in **Kernel.php** in **App/Http** folder, open and find this:
```php
protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
    ];
```
- Add a our middleware after the guest
```php
protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'editor' => \App\Http\Middleware\EditorMiddleware::class,
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
    ];
```
- Now we need to call this middleware, for that go to **EditorController** and update the **constructor** as follows
```php
public function __construct()
    {
        $this->middleware('auth:admin');
        $this->middleware('editor');
    }
```
- Now when you logged in as admin and try to access the editor dashboard i.e **admin/editor** , you will be redirected into the welcome page.
- but when you logged in as editor and try to access the admin dashboard i.e. **admin/home,** you can still access, for solving this create a new middleware for admin 
- repeat above steps and final files will look like this
1. **AdminMiddleware.php**
```php
use Auth;
public function handle($request, Closure $next)
    {
        
        foreach (Auth::user()->role as $role) {
            
            if ($role->name == "admin") {
                
                return $next($request);
            }
        }

        return redirect('/');

    }
```
1. **Kernel.php**
```php
protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
        'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
        'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
        'can' => \Illuminate\Auth\Middleware\Authorize::class,
        'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
        'editor' => \App\Http\Middleware\EditorMiddleware::class,
        'admin' => \App\Http\Middleware\AdminMiddleware::class,
        'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
        'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
        'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
    ];
```
1. **AdminController.php**
```php
public function __construct()
    {
        $this->middleware('auth:admin');
        $this->middleware('admin');
    }
```
- Now everything is ready, let check request lifecycle
1. Logged in as **Admin** and try to access **admin/editor -** redirects to **welcome**
1. From the **welcome** page try to access the **/admin** - redirect to **admin dashboard**, bcoz of the **RedirectIfAuthenticated.php** middleware.
1. logout from **Admin**
1. Logged in as **Editor** and try to access **admin/home** - redirects to **welcome**
1. From the **welcome** page try to access the **/admin** - redirect to **welcome** instead of **editor dashboard**, bcoz of the editor case is not defined in **RedirectIfAuthenticated.php** middleware.
- So lets open the RedirectIfAuthenticated.php middleware
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
-     update as follows
```php
public function handle($request, Closure $next, $guard = null)
    {   
        switch ($guard) {

            case 'admin':
            
                if (Auth::guard($guard)->check()) {
            
                    foreach (Auth::guard($guard)->user()->role as $role) {
            
                        if ($role->name == "admin") {
                            
                            return redirect('admin/home');

                        } elseif ($role->name == "editor") {
                
                            return redirect('admin/editor');

                        }
                    }
                    
                    //return redirect('admin/home');
            
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
-     Now its solved!
---
## Create a view which can view both admin and editor
- open **web.php** and add a new route for the **test view**
```php
Route::get('admin/test', 'EditorController@test');
```
- create a new view called **test.blade.php** in **Admin** folder and copy below code and paste it into it
```php
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">Admin cum Editor View</div>

                <div class="card-body">
                    @if (session('status'))
                        <div class="alert alert-success" role="alert">
                            {{ session('status') }}
                        </div>
                    @endif

                    You can access this page as both admin and editor!
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```
- Now open the **EditorController** and add the new view
```php
/**
     * Show the test page for both admin and editor.
     *
     * @return \Illuminate\Http\Response
     */
    public function test()
    {

        return view('admin.test');

    }
```
-     also update the middleware as follows,
```php
public function __construct()
    {
        $this->middleware('auth:admin');
        $this->middleware('editor', ['except' => 'test']);
    }
```
- If the **exception** is not added, **Admin** can't access the **test view** only **Editor** can view.
- Also only the authenticated **admin user** can view the **test view**, bcoz of the **auth middleware** declared above
- also the **normal user**s can't access the test view, bcoz of **auth:admin middleware**
---
- Now if Logged in as user and try to access the admin login i.e **/admin,** we can see the admin login form
- Now if Logged in as admin and try to access the user login i.e **/login**, we can see the user login form
- for solving this, go to **LoginController.php** in **Admin** folder and update the middleware as follows
```php
public function __construct()
    {
        $this->middleware('guest:admin')->except('logout');
        //for restricting the  admin to access user login after authentication
        $this->middleware('guest')->except('logout');
    }
```
- Now open the **LoginController.php** in **Auth** folder and update the middleware as follows
```php
public function __construct()
    {
        $this->middleware('guest')->except('logout');
        //for restrictiing the users to access the admin login after logged in as user
        $this->middleware('guest:admin')->except('logout');
    }
```
- Now the problem is solved
- **Finished!**