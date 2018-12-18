# Laravel - Multi Auth : 1
- Go to **App/Http/Controllers/Auth** folder and copy all controller files and paste into new folder called **App/Http/Controllers/Admin** folder
- Now the **Admin**  folder wiil have the following files
1. ForgotPasswordController.php
1. LoginController.php
1. RegisterController.php
1. ResetPasswordController.php
1. VerificationController.php
- Lets open all those files and find the **namespace** 
```php
namespace App\Http\Controllers\Auth;
```
- Update to new **namespace** 
```php
namespace App\Http\Controllers\Admin;
```
- Now go to **resources/views/auth** folder and copy all view files and paste into the new folder called **resources/views/admin** folder
- Now the **admin** folder will have the following files
1. email.blade.php in **passwords** directory
1. reset.blade.php in **passwords** directory
1. login.blade.php
1. register.blade.php
1. verify.blade.php
- Now open **login.blade.php** in **admin** folder and update as follows
```php
<div class="card-header">{{ __('Login') }}</div> 
Update to
<div class="card-header">{{ __('Admin Login') }}</div>
```
- next
```php
<form method="POST" action="{{ route('login') }}">
Update to
<form method="POST" action="{{ route('admin.login') }}">
```
- next
```php
<a class="btn btn-link" href="{{ route('password.request') }}">
Update to
<a class="btn btn-link" href="{{ route('admin.password.request') }}">
```
- Now open **email.blade.php** in **admin/passwords** folder and update as follows
```php
<div class="card-header">{{ __('Reset Password') }}</div>
Update to
<div class="card-header">{{ __('Admin Reset Password') }}</div>
```
- next
```php
<form method="POST" action="{{ route('password.email') }}">
Update to
<form method="POST" action="{{ route('admin.password.email') }}">
```
- Now open **email.blade.php** in **admin/passwords** folder and update as follows
```php
<div class="card-header">{{ __('Reset Password') }}</div>
Update to
<div class="card-header">{{ __('Admin Reset Password') }}</div>
```
- next
```php
<form method="POST" action="{{ route('password.update') }}">
Update to
<form method="POST" action="{{ route('admin.password.update') }}">
```
- Now create new **home.blade.php** foe **admin**s in admin folder paste the following content to the file
```php
//home.blade.php
@extends('layouts.app')

@section('content')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">Admin Dashboard</div>

                <div class="card-body">
                    @if (session('status'))
                        <div class="alert alert-success" role="alert">
                            {{ session('status') }}
                        </div>
                    @endif

                    You are logged in as Admin!
                </div>
            </div>
        </div>
    </div>
</div>
@endsection
```

---
- Now  create **routes** for the following admin functions, for that open **web.php** and add following lines of code
```php
//--------- Admin routes -------------------//

Route::get('admin/home', 'AdminController@index');

Route::get('admin', 'Admin\LoginController@showLoginForm')->name('admin.login');
Route::post('admin', 'Admin\LoginController@login');
Route::post('admin-password/email', 'Admin\ForgotPasswordController@sendResetLinkEmail')->name('admin.password.email');
Route::get('admin-password/reset', 'Admin\ForgotPasswordController@showLinkRequestForm')->name('admin.password.request');
Route::post('admin-password/reset', 'Admin\ResetPasswordController@reset')->name('admin.password.update');
Route::get('admin-password/reset/{token}', 'Admin\ResetPasswordController@showResetForm')->name('admin.password.reset');
```

---
- Now lets create the **AdminController.php**  in **App/Http/Controllers** folder and paste following lines of code.
```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class AdminController extends Controller
{
    /**
     * Create a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
       // $this->middleware('auth:admin');
    }

    /**
     * Show the application dashboard.
     *
     * @return \Illuminate\Http\Response
     */
    public function index()
    {
        return view('admin.home');
    }
}
```

---
- Lets create **guard** for **admins,** just open **auth.php** in **config** folder
- creating the **admin** **guard**
```php
 'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],

        'admin' => [
            'driver' => 'session',
            'provider' => 'admins',
        ],

        'api' => [
            'driver' => 'token',
            'provider' => 'users',
        ],
    ],
```
-     creating the **admins provider**
```php
'providers' => [
        'users' => [
            'driver' => 'eloquent',
            'model' => App\User::class,
        ],

        'admins' => [
            'driver' => 'eloquent',
            'model' => App\Admin::class,
        ],

        // 'users' => [
        //     'driver' => 'database',
        //     'table' => 'users',
        // ],
    ],
```
-     set the **passwords reset config**
```php
'passwords' => [
        'users' => [
            'provider' => 'users',
            'table' => 'password_resets',
            'expire' => 60,
        ],

        'admins' => [
            'provider' => 'admins',
            'table' => 'password_resets',
            'expire' => 60,
        ],
    ],
```

---
- create a **Admin model** and paste following contents into **Admin.php**
```php
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class Admin extends Authenticatable
{
    use Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'lastname', 'email', 'password',
    ];

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password', 'remember_token',
    ];
}
```
- Open **LoginController.php** update the following line
```php
protected $redirectTo = 'admin/home';
```
- and update new middleware **guest as admin**
```php
public function __construct()
{
    $this->middleware('guest:admin')->except('logout');
}
```
- update new middleware **guest as admin** in **ForgotPasswordController** also
```php
//ForgotPasswordController.php
public function __construct()
    {
        $this->middleware('guest:admin');
    }
```
- update new middleware **guest as admin** in **RegisterController** also
```php
//RegisterController.php
public function __construct()
    {
        $this->middleware('guest:admin');
    }
```
- also in **RegisterController**
```php
protected $redirectTo = 'admin/home';
```
- update new middleware **guest as admin** in **ResetPasswordController** also
```php
public function __construct()
    {
        $this->middleware('guest:admin');
    }
```
-    also in **ResetPasswordController**
```php
protected $redirectTo = 'admin/home';
```

---
- Now create **migration table** for **admins,** run the following command
```php
php artisan make:migration create_admins_table --create=Admins
```
- Lets open **create_admins_table** in **database/migrations** folder and update the Schema as folows
```php
<?php

use Illuminate\Support\Facades\Schema;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;

class CreateAdminsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('admins', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('admins');
    }
}
```
- now lets run the migration command : 
```php
php artisan migrate
```
- Now lets create new **admin** via the **database GUI** (e.g: phpMyAdmin) . i.e using the **insert** option insert the following data
1. name
1. email
1. password - **for demo purpose copy the hashed password from the users table**
- **Finished!, Let see in part 2**
