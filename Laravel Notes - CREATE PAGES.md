# Laravel Notes - CREATE PAGES
### Creating new pages
- create a controller (e.g: PagesController.php) and place in app/Http/controllers
- In controller class update as follows:
```html
<?php namespace App\Http\Controllers;
use App\Http\Requests;
use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
class PagesController extends Controller {
    public function home()
    {
        return view('welcome');
    }
}
```
- Aritsan command for creating controller
 This controller contains several action like index, create, destroy etc;
             `php artisan make: controller controllerName`
 plain controller file
              `php artisan make controller controllerName --plain`
-   change web.php for updating routes to the new pages
             `Route::get('/pageName', 'controllerName@pageName');`
-  create view to this pages, create viewFileName.blade.php and update with html content, save to
        resources/views directory
        
### BLADE TEMPLATE
```html
@if ($product == 1)
    {!! $product->name !!}
@else
    There is no product!
@endif
```
Equivalent PHP code:
```html
if ($product ==1) {
    echo $product->name;
} else {
    echo("There is no product!");
}
```
