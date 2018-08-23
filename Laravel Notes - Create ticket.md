# Laravel Notes - Create ticket
- Create a view called create.blade.php place in resources/views/tickets/
- Create **TicketsController** : `php artisan make : controller TicketsController`
- Update PagesController 
```php
public function create()
{
    return view('tickets.create');
}
```
- Update **web.php** file : `Route::get('/url', 'TicketsController@create');`
- Use **Request** to validate the form : `php artisan make : request TicketFormRequest`
- In **TicketFormRequest.php** , there will be 2 methods:
1. **authorize** method - returns false or true means no one can able to perform the request or able to perform resptvly.
1. **rules** method - define our validation rules. [learn more](http://laravel.com/docs/master/validation#available-validation-rules)
```php
public function authorize()
{
    return false;
}
public function rules()
{
    return [
        'title' => 'required|min:3',
        'content'=> 'required|min:10',
    ];
}
```
- Update **web.php :** `Route::post('/url', 'TicketsController@store');`
- Update **TicketsController.php :**
```php
public function store(TicketFormRequest $request)
{
    return $request->all();
}
```
- Laravel requires us to type-hint the IlluminateHttpRequest class on our controller constructor to obtain an instance of the current HTTP request. So put this line `use App\Http\Requests\TicketFormRequest;` **above** `class TicketsController extends Controller`
- Add a hidden token field below your form opening tag. Since Laravel now requires a token to be sent when using the POST method. otherwise it throws an error : **TokenMismatchException**
```php
<form class="form-horizontal" method="post">
    <input type="hidden" name="_token" value="{!! csrf_token() !!}">
```
-     to display the errors when the users don’t fill the form or the form is not valid. **Find:**
            `<form class="form-horizontal" method="post">`     **add below:**
```php
@foreach ($errors->all() as $error)
    <p class="alert alert-danger">{{ $error }}</p>
@endforeach
```
> Basically, if the validator fails, Laravel will store all errors in the session. We can easily access the errors via **$errors** object.
## Insert data into the database
- Put this line `use App\Ticket;` above the class name of **TicketsController.php**, this tells we want to use **Ticket model** in this class to store the form data.
- Update the **store** action :
```php
public function store(TicketFormRequest $request)
{
    $slug = uniqid();
    $ticket = new Ticket(array(
        'title' => $request->get('title'),
        'content' => $request->get('content'),
        'slug' => $slug
    ));
    $ticket->save();
    return redirect('/contact')->with('status', 'Your ticket has been created! I\
    ts unique id is: '.$slug);
}
```
1. **uniqid()** function to generate a unique ID based on the microtime. You may use **md5()** function to generate the slugs or create your custom slugs.
- Open Ticket Model i.e. Ticket.php  place this for avoiding mass assignment, [more at](http://laravel.com/docs/master/eloquent#mass-assignment)
```php
class Ticket extends Model
{
    protected $fillable = ['title', 'content', 'slug', 'status', 'user_id'];
}
```
- **$fillable** property make the columns mass assignable.
- Alternatively, you may use the **$guarded** property to make all attributes mass assignable except for your chosen attributes. For example, I use the id column here: `protected $guarded = ['id'];`
> Note: You must use either $fillable or $guarded.
- update the **tickets/create.blade.php** view to display the status message **find**:
```php
<form class="form-horizontal" method="post">
    @foreach ($errors->all() as $error)
        <p class="alert alert-danger">{{ $error }}</p>
    @endforeach
```
    **Add below:**
```php
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```
