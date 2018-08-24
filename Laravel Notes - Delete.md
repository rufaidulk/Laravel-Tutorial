# Laravel Notes - Delete
- `Route::post('/ticket/{slug?}/delete','TicketsController@destroy');`
> Note: You may use the GET method here.
- Update destroy action : 
```php
public function destroy($slug)
{
    $ticket = Ticket::whereSlug($slug)->firstOrFail();
    $ticket->delete();
    return redirect('/tickets')->with('status', 'The ticket '.$slug.' has been d\
        eleted!');
}
```
- redirect users to the all tickets page. Let’s update the index.blade.php to display the status message, **Find:**
```php
<div class="panel-heading">
    <h2> Tickets </h2>
</div>
```
- **Add below:**
```php
@if (session('status'))
    <div class="alert alert-success">
        {{ session('status') }}
    </div>
@endif
```
- in order to remove the ticket, all we need to do is create a form to submit a delete request. Open show.blade.php and **find:**
```php
<a href="{!! action('TicketsController@edit', $ticket->slug) !!}" class="btn btn\
-info">Edit</a>
<a href="#" class="btn btn-info">Delete</a>
```
- **Update to:**
```php
<a href="{!! action('TicketsController@edit', $ticket->slug) !!}" class="btn btn\
-info pull-left">Edit</a>
    <form method="post" action="{!! action('TicketsController@destroy', $ticket->slu\
        g) !!}" class="pull-left">
        <input type="hidden" name="_token" value="{!! csrf_token() !!}">
        <div class="form-group">
            <div>
                <button type="submit" class="btn btn-warning">Delete</button>
            </div>
        </div>
    </form>
<div class="clearfix"></div>
```
