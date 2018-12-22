# Laravel - Multi Auth : 5 - User login status
- Go to **phpMyAdmin** and create a new column called **status** with **boolean type** after the lastname column
- update **users migration** as follows
```php
public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('lastname');
            $table->boolean('status');
            $table->string('email')->unique();
            $table->timestamp('email_verified_at')->nullable();
            $table->string('password');
            $table->rememberToken();
            $table->timestamps();
        });
    }
```
-     update User model for adding default value for status while registering new users
```php
/**
     * Default value for the status of the user, i.e 1 or active and 0 for inactive
     *
     * @var array
     */
    protected $attributes = [
        'status' => 1,
    ];
```
- open **LoginController** in **Auth** folder and find this line:
```php
use AuthenticatesUsers;
```
- open this **AuthenticatesUsers.php** in **vendor/laravel/framework/src/Illuminate/foundation/Auth** folder
- copy the **credentials()** method and paste it into the **LoginController.php** in **Auth** folder, update as follows
```php
/**
     * Get the needed authorization credentials from the request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    protected function credentials(Request $request)
    {
        return $request->only($this->username(), 'password');
    }
```
- Update as follows
```php
/**
     * Get the needed authorization credentials from the request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return array
     */
    protected function credentials(Request $request)
    {
        //return $request->only($this->username(), 'password');
        
        return ['email'=> $request->{$this->username()}, 'password' => $request->password, 'status' => '1'];
    }
```
-  Now if the status is **0** users can't login, 
- finished!