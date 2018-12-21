# Laravel - Multi Auth : 3 - Reset Password
- Lets open **ForgotPasswordController.php** and find this line:
```php
use SendsPasswordResetEmails;
```
- Open this file **SendsPasswordResetEmails.php** and find the **showLinkRequestForm()** method, copy and overwrite this method in **ForgotPasswordController.php** as follows:
```php
/**
     * Display the form to request a password reset link.
     *
     * @return \Illuminate\Http\Response
     */
    public function showLinkRequestForm()
    {
        return view('admin.passwords.email');
    }
```
- Now when you click the **forgot password link**  in **http://app-name.test/admin ,** You will be redirected to admin reset password page
- Let open the **web.php** and look at the working password reset
```php
//--------- Admin routes -------------------//

Route::get('admin/home', 'AdminController@index');

Route::get('admin', 'Admin\LoginController@showLoginForm')->name('admin.login');
Route::post('admin', 'Admin\LoginController@login');

//-------- Admin password reset ------//

Route::get('admin-password/reset', 'Admin\ForgotPasswordController@showLinkRequestForm')->name('admin.password.request');
Route::post('admin-password/email', 'Admin\ForgotPasswordController@sendResetLinkEmail')->name('admin.password.email');
Route::get('admin-password/reset/{token}', 'Admin\ResetPasswordController@showResetForm')->name('admin.password.reset');
Route::post('admin-password/reset', 'Admin\ResetPasswordController@reset')->name('admin.password.update');
```
1. Initially reset link request form will shown when click the forgot password link, i.e  route **admin-password/reset** shows the **showLinkRequestForm()** method in **ForgotPasswordController.php** which is a **get** request.
1. when submit the email id corresponds to the admin account, it calles **post** request, i.e route **admin-password/email** calls the **sendResetLinkEmail()**  method in the **ForgotPasswordController.php.**
1. Then we will recieve the reset mail and click the reset link, which redirects to the route **admin-password/reset/{token}** which shows the reset form by calling the **showResetForm()** method in **ResetPasswordController.php** which is a **get** request.
1. After submitting new password and click submit, this invokes the last route **admin-password/reset** which is a **post request** and call the **reset()** method in the **ResetPasswordController.php**

---
- Now lets do the first step, i.e click the **forgot password link** which shows your the form for requesting reset link, here we can see the admin reset form bcoz we have overwritten **showLinkRequestForm()** in **ForgotPasswordController.php.**
- Now when you enter the email address and submit the form, its call the STEP 2, i.e **sendResetLInkEmail(),** just look the this method which is located in **SendsPasswordResetEmails.php**
```php
 /**
     * Send a reset link to the given user.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\RedirectResponse|\Illuminate\Http\JsonResponse
     */
    public function sendResetLinkEmail(Request $request)
    {
        $this->validateEmail($request);

        // We will send the password reset link to this user. Once we have attempted
        // to send the link, we will examine the response then see the message we
        // need to show to the user. Finally, we'll send out a proper response.
        $response = $this->broker()->sendResetLink(
            $request->only('email')
        );

        return $response == Password::RESET_LINK_SENT
                    ? $this->sendResetLinkResponse($request, $response)
                    : $this->sendResetLinkFailedResponse($request, $response);
    }
```
-     follow each steps which explains each lines in the **sendResetLinkMail()**
###     Step 1
- Initially it will validate the submitted email by calling the **validateEmail()** method which is located in the same file
```php
/**
     * Validate the email for the given request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return void
     */
    protected function validateEmail(Request $request)
    {
        $request->validate(['email' => 'required|email']);
    }
```
### Step 2
- Then it will call the **password ****broker()**** ,**which call the default broker() but we have another broker() for admins , i.e look at the **Auth.php** in **config** folder
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
- Therfore, we have overwrite the password **broker()** in the **ForgotPasswordController**, for that copy the default **broker()** from the **SendsPasswordResetEmails.php**
```php
/**
     * Get the broker to be used during password reset.
     *
     * @return \Illuminate\Contracts\Auth\PasswordBroker
     */
    public function broker()
    {
        return Password::broker();
    }
```
- Update as follows and paste it into the **ForgotPasswordController**
```php
/**
     * Get the broker to be used during password reset.
     *
     * @return \Illuminate\Contracts\Auth\PasswordBroker
     */
    public function broker()
    {
        return Password::broker('admins');
    }
```
>     This new broker() will use the admins provider powered by elqouent and uses Admins Model.                i.e it will check the submitted email is present in the admin table which means admin is present or      
 not before sending reset email
- Now we submit the form we will get reset link via email, but when we click the link we will get reset password of users instead of admins. for solving this follow below steps.
---
### Step 3
- open **ResetPasswordController.php** find this line:
```php
use ResetsPasswords;
```
- open this **ResetPasswords.php** in **vendor/laravel/framework/src/Illuminate/Foundation/Auth** folder and copy the **showResetForm()** method and paste into **ResetPasswordController.php**
- update the method as follows
```php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Password;

/**
     * Display the password reset view for the given token.
     *
     * If no token is present, display the link request form.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  string|null  $token
     * @return \Illuminate\Contracts\View\Factory|\Illuminate\View\View
     */
    public function showResetForm(Request $request, $token = null)
    {
        return view('admin.passwords.reset')->with(
            ['token' => $token, 'email' => $request->email]
        );
    }
```
- For getting above reset form via link do as follows.
### Step 4
- You can see the sendResetLink() method after the broker() 
```php
$response = $this->broker()->sendResetLink(
            $request->only('email')
        );
```
- open the **sendResetLink()** in **vendor/laravel/framework/src/Illuminate/Auth/Passwords/PasswordBroker.php**
```php
/**
     * Send a password reset link to a user.
     *
     * @param  array  $credentials
     * @return string
     */
    public function sendResetLink(array $credentials)
    {
        // First we will check to see if we found a user at the given credentials and
        // if we did not we will redirect back to this current URI with a piece of
        // "flash" data in the session to indicate to the developers the errors.
        $user = $this->getUser($credentials);

        if (is_null($user)) {
            return static::INVALID_USER;
        }

        // Once we have the reset token, we are ready to send the message out to this
        // user with a link to reset their password. We will then redirect back to
        // the current URI having nothing set in the session to indicate errors.
        $user->sendPasswordResetNotification(
            $this->tokens->create($user)
        );

        return static::RESET_LINK_SENT;
    }
```
-     open the **sendPasswordResetNotification()** in **vendor/laravel/framework/src/Illuminate/Auth/Passwords/CanResetPassword.php** and find this line
```php
use Illuminate\Auth\Notifications\ResetPassword as ResetPasswordNotification;
```
- open the **ResetPassword.php** in **vendor/laravel/framework/src/Illuminate/Auth/Notifications** folder
- You can see the below method, which sends email
```php
/**
     * Build the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        if (static::$toMailCallback) {
            return call_user_func(static::$toMailCallback, $notifiable, $this->token);
        }

        return (new MailMessage)
            ->subject(Lang::getFromJson('Reset Password Notification'))
            ->line(Lang::getFromJson('You are receiving this email because we received a password reset request for your account.'))
            ->action(Lang::getFromJson('Reset Password'), url(config('app.url').route('password.reset', $this->token, false)))
            ->line(Lang::getFromJson('If you did not request a password reset, no further action is required.'));
    }
```
- This is ****Notification**** which is default for auth for users, so we need to create custome Notification for admins, for that run:
```php
php artisan make:notification AdminResetPasswordNotification
```
- open the **AdminResetPasswordNotification.php** in **App/Notification** folder, where you can see the below **toMail()** 
```php
/**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
                    ->line('The introduction to the notification.')
                    ->action('Notification Action', url('/'))
                    ->line('Thank you for using our application!');
    }
```
-     we need to update this as follows
```php

use Illuminate\Support\Facades\Lang;

/**
     * Create a new notification instance.
     *
     * @return void
     */
    public function __construct($token)
    {
        $this->token = $token;
    }
/**
     * Get the mail representation of the notification.
     *
     * @param  mixed  $notifiable
     * @return \Illuminate\Notifications\Messages\MailMessage
     */
    public function toMail($notifiable)
    {
        return (new MailMessage)
            ->subject(Lang::getFromJson('Reset Password Notification'))
            ->line(Lang::getFromJson('You are receiving this email because we received a password reset request for your account.'))
            ->action(Lang::getFromJson('Admin Reset Password'), url(config('app.url').route('admin.password.reset', $this->token, false)))
            ->line(Lang::getFromJson('If you did not request a password reset, no further action is required.'));
    }
```
- we have customized email and add the **token** in the constructor for using in the **toMail()** 
- Look at the **CanResetPassword.php** in **vendor/laravel/framework/src/Illuminate/Auth/Passwords** folder you can see a method like this:
```php
/**
     * Send the password reset notification.
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new ResetPasswordNotification($token));
    }
```
- we need to overwrite like below:
```php

use App\Notifications\AdminResetPasswordNotification;
/**
     * Send the password reset notification.
     *
     * @param  string  $token
     * @return void
     */
    public function sendPasswordResetNotification($token)
    {
        $this->notify(new AdminResetPasswordNotification($token));
    }
```
- we are using custom notification and paste into the **Admin** model.
- Now we will recieve the correct email but when we try to reset, it will reset the users instead of admins. i.e it will look into the users table.
---
### Step 5
- Go to **ResetPasswordController.php** find this line:
```php
use ResetsPasswords;
```
- open **ResetPasswords.php** in **vendor/laravel/framework/src/Illuminate/foundation/Auth** folder
- you can find the default **guard() and broker()** for resetting the password which is used by the users
- we need to overwrite these for admins, so copy the **guard() and broker()** and paste into **ResetPasswordController.php** and update as follows:
```php
/**
     * Get the broker to be used during password reset.
     *
     * @return \Illuminate\Contracts\Auth\PasswordBroker
     */
    public function broker()
    {
        return Password::broker('admins');
    }

    /**
     * Get the guard to be used during password reset.
     *
     * @return \Illuminate\Contracts\Auth\StatefulGuard
     */
    protected function guard()
    {
        return Auth::guard('admin');
    }
```
-     **Now the reset password for the admin is set!**