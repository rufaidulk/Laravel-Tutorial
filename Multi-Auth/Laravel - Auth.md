# Laravel - Auth
- To create the inbuilt authentication in laravel run the following command:
```php
php artisan make:auth
php artisan migrate //migrate the auth tables
```
- Every **route**  of authentication is controlled by **route()** function in  **web.php**
```php
Auth::routes(); 
```
- **routes()** function is defined in **Auth.php** in  **vendor/laravel/framework/src/Illuminate/Support/Facades/** folder
```php
public static routes(array $options = [])
{
    static::$app->make('router')->auth($options);
}
```
- In the **routes()** function an **auth()**  function is there, which is defined in **Router.php** in **vendor/laravel/framework/src/Illuminate/Routing/** folder
```php
public function auth(array $options = [])
    {
        // Authentication Routes...
        $this->get('login', 'Auth\LoginController@showLoginForm')->name('login');
        $this->post('login', 'Auth\LoginController@login');
        $this->post('logout', 'Auth\LoginController@logout')->name('logout');

        // Registration Routes...
        if ($options['register'] ?? true) {
            $this->get('register', 'Auth\RegisterController@showRegistrationForm')->name('register');
            $this->post('register', 'Auth\RegisterController@register');
        }

        // Password Reset Routes...
        if ($options['reset'] ?? true) {
            $this->resetPassword();
        }

        // Email Verification Routes...
        if ($options['verify'] ?? false) {
            $this->emailVerification();
        }
    }
```
-     this is where all routes related to **auth** is defined.
---
- Now open **LoginController.php** in **Controller/Auth** folder
```php
protected $redirectTo = '/home';

    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('guest')->except('logout');
    }
```
- above line of code decides where user is redirected to if user is logged in.
- for viewing **guest** middleware shown above, go to **Kernel.php**
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
-     **guest** middleware is in **RedirectIfAuthenticated.php,** 
```php
public function handle($request, Closure $next, $guard = null)
    {
        if (Auth::guard($guard)->check()) {
            return redirect('/home');
        }

        return $next($request);
    }
```
-     above **redirect()**  method specifies that, when user logged in and try to access the **/login**  route it will redirect into **/home** route. this is done by the **Auth::guard()**
---
- Now check the routes using the command 
```php
php artisan route:list
```

![Image](https://www.dropbox.com/s/h7xz3perrk6xnb1/S1Mf2A2yN_HkGSAeTJE.png?dl=1)
- You can see the **LoginController** have function like **showLoginForm, login, logout** . they cannot be seen in the **LoginController.php**
- for that go to **LoginController.php** and you can see **AuthenticatesUsers.php** class, above mentioned methods are defined in this class.
```php
use AuthenticatesUsers;
```
- when user submits a login credentials , forms gets login route, which will call the **login()** in **AuthenticatesUsers.php** class
```php
<form method="POST" action="{{ route('login') }}">
```
- let open the **login** method in **AuthenticatesUsers.php**
```php
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
1. first it will validate the request, `$this->validateLogin($request);` look at the **validateLogin(**$request) method
```php
protected function validateLogin(Request $request)
    {
        $request->validate([
            $this->username() => 'required|string',
            'password' => 'required|string',
        ]);
    }
    // kusername methaod
    public function username()
    {
        return 'email';
    }
```
- username and password is required and must be string, here username is email id
- you can change email by name or something else for login
### Login with name
- change the **username()** method and update it as follows
```php
public function username()
    {
        return 'name'; //since the name of the field in user table
    }
```
- go to **login.blade.php**  and replace email with name as follows
```php
<div class="form-group row">
    <label for="name" class="col-md-4 col-form-label text-md-right">{{ __('Name') }}</label>
        <div class="col-md-6">
            <input id="name" type="name" class="form-control{{ $errors->has('name') ? ' is-invalid' : '' }}" name="name" value="{{ old('name') }}" required autofocus>

                @if ($errors->has('name'))
                    <span class="invalid-feedback" role="alert">
                        <strong>{{ $errors->first('name') }}</strong>
                    </span>
                @endif
        </div>
</div>
```
- now you can login using name of the user.
1. nest you can see a line `$this->hasTooManyLoginAttempts($request)`  , this will decided no: of incorrect login attempts and   how much time should delay the login.
1. open the **hasTooManyLoginAttempts** method in **ThrottlesLogin.php**  in **vendor/laravel/framework/src/Illuminate/Foundation/Auth** folder.
```php
public function maxAttempts()
    {
        return property_exists($this, 'maxAttempts') ? $this->maxAttempts : 5;
    }

    /**
     * Get the number of minutes to throttle for.
     *
     * @return int
     */
    public function decayMinutes()
    {
        return property_exists($this, 'decayMinutes') ? $this->decayMinutes : 1;
    }
```
- you can change maximum no of login attempts and time to delay in minutes using above two methods which is in the **ThrottlesLogin.php**
---
### **Login with email instead of password**
- You can see the **attemptLogin()** method in **AuthenticatesUsers.php**
```php
protected function attemptLogin(Request $request)
    {
        return $this->guard()->attempt(
            $this->credentials($request), $request->filled('remember')
        );
    }
```
-     open the **credentials()** method mentioned above in the same file
```php
protected function credentials(Request $request)
    {
        return $request->only($this->username(), 'password');
    }
```
-     replace **password** with **email**
- also update **validateLogin()** method in the same file, replace password with email
```php
protected function validateLogin(Request $request)
    {
        $request->validate([
            $this->username() => 'required|string',
            'password' => 'required|string',
        ]);
    }
```
- Now open **EloquentUserProvider.php** in **vendor/laravel/framework/src/Illuminate/Auth** folder and find the **validateCredentials()** method 
```php
public function validateCredentials(UserContract $user, array $credentials)
    {
        $plain = $credentials['password'];

        return $this->hasher->check($plain, $user->getAuthPassword());
    }
```
-     replace password with email and comment the second line which converts password into hash. since we don't need to convert the input email to hash code.so update file look like this.
```php
public function validateCredentials(UserContract $user, array $credentials)
    {
        $plain = $credentials['email'];

        return $plain;//$this->hasher->check($plain, $user->getAuthPassword());
    }
```
-     Now go to **login.blade.php**  and replace password with email.. Now you can login with email instead of password.
---
- Let see the **RegisterController.php** 
```php
protected $redirectTo = '/home';
```
- above line decides where should user go after the registration
- we can't access the **register** route after registering bcoz of the **guest middleware**
```php
public function __construct() 
{
    $this->middleware('guest');
}
```
- All the routes for **RegisterController** is defined in **RegistersUsers.php** in **vendor/laravel/framework/src/Illuminate/Foundation/Auth** folder

---
### Add Lastname to Register Form
- Initially go to **register.blade.php** and add new field for lastname
```php
<div class="form-group row">
    <label for="lastname" class="col-md-4 col-form-label text-md-right">{{ __('Last Name') }}</label>
    <div class="col-md-6">
        <input id="lastname" type="text" class="form-control{{ $errors->has('lastname') ? ' is-invalid' : '' }}" name="lastname" value="{{ old('lastname') }}" required autofocus>
                @if ($errors->has('lastname'))
                    <span class="invalid-feedback" role="alert">
                        <strong>{{ $errors->first('lastname') }}</strong>
                    </span>
                @endif
    </div>
</div>
```
- then open **create_user_table** and add a field for lastname
```php
public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('lastname'),
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }
```
- now run **`php artisan migrate:refresh`**
- Now open the **RegisterController.php** update the **validator()** and  **create()** methods as follows
```php
protected function validator(array $data)
    {
        return Validator::make($data, [
            'name' => ['required', 'string', 'max:255'],
            'lastname' => ['required', 'string', 'max:255'],
            'email' => ['required', 'string', 'email', 'max:255', 'unique:users'],
            'password' => ['required', 'string', 'min:6', 'confirmed'],
        ]);
    }
    
    protected function create(array $data)
    {
        return User::create([
            'name' => $data['name'],
            'lastname' => $data['lastname'],
            'email' => $data['email'],
            'password' => Hash::make($data['password']),
        ]);
    }
    
```
-     Then open **User** model and add lastname to **$fillable** property for mass assignment
```php
protected $fillable = [
        'name', 'lastname', 'email', 'password',
    ];
```
-     Its done! 
---
- For showing the full name on the **home view** just open **app.blade.php** and update as follows
```php
<a id="navbarDropdown" class="nav-link dropdown-toggle" href="#" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false" v-pre>
        {{ Auth::user()->name.' '.Auth::user()->lastname }} <span class="caret"></span>
</a>
```
- **Finished!**