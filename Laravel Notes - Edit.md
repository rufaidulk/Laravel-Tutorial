# Laravel Notes - Edit
- `Route::get('/ticket/{slug?}/edit','TicketsController@edit');`
- Update edit action : 
```php
public function edit($slug)
{
    $ticket = Ticket::whereSlug($slug)->firstOrFail();
    return view('tickets.edit', compact('ticket'));
}
```
- edit view :
```php
@extends('master')
@section('title', 'Edit a ticket')
@section('content')
<div class="container col-md-8 col-md-offset-2">
    <div class="well well bs-component">
        <form class="form-horizontal" method="post">
            @foreach ($errors->all() as $error)
                <p class="alert alert-danger">{{ $error }}</p>
            @endforeach
            @if (session('status'))
                <div class="alert alert-success">
                    {{ session('status') }}
                </div>
            @endif
            <input type="hidden" name="_token" value="{!! csrf_token() !!}">
            <fieldset>
                <legend>Edit ticket</legend>
                <div class="form-group">        
                    <label for="title" class="col-lg-2 control-label">Title</label>
                    <div class="col-lg-10">
                        <input type="text" class="form-control" id="title" n\
                            ame="title" value="{!! $ticket->title !!}">
                    </div>
                </div>
                <div class="form-group">
                    <label for="content" class="col-lg-2 control-label">Content</label>
                    <div class="col-lg-10">
                        <textarea class="form-control" rows="3" id="content"\
                            name="content">{!! $ticket->content !!}</textarea>
                    </div>
                </div>
                <div class="form-group">
                    <label>
                        <input type="checkbox" name="status" {!! $ticket->st\
                            atus?"":"checked"!!} > Close this ticket?
                    </label>
                </div>
                <div class="form-group">
                    <div class="col-lg-10 col-lg-offset-2">
                        <button class="btn btn-default">Cancel</button>
                        <button type="submit" class="btn btn-primary">Update</button>
                    </div>
                </div>
            </fieldset>
        </form>
    </div>
</div>
@endsection
```
- `{!! $ticket->status?"":"checked"!!}` : If the status is 1 (pending), we display nothing, the checkbox is not checked. If the status is 0 (answered) , we display a checked attribute, the checkbox is checked.
- open the show view, update the edit button as follows: `<a href="{!! action('TicketsController@edit', $ticket->slug) !!}" class="btn btn-info">Edit</a>`
- We use the action helper again! When you click on the edit button, you should be able to access the edit form:
- Need to use POST method to submit the form. `Route::post('/ticket/{slug?}/edit','TicketsController@update');`
- Use update action:
```php
public function update($slug, TicketFormRequest $request)
{
    $ticket = Ticket::whereSlug($slug)->firstOrFail();
    $ticket->title = $request->get('title');
    $ticket->content = $request->get('content');
    if($request->get('status') != null) {
        $ticket->status = 0;
    } else {
        $ticket->status = 1;
    }
    $ticket->save();
    return redirect(action('TicketsController@edit', $ticket->slug))->with('stat\
        us', 'The ticket '.$slug.' has been updated!');
}
```
1. This is how we can check if the users select the status checkbox or not:
```php
if($request->get('status') != null) {
    $ticket->status = 0;
} else {
    $ticket->status = 1;
}
```
1. redirect users to the ticket page with a status message: `return redirect(action('TicketsController@edit', $ticket->slug))->with('status','The ticket '.$slug.' has been updated!');`