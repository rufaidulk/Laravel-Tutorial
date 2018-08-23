# Laravel Notes - Model
Using **eloquent ORM** to create, edit, delete table without using any sql 
- Create a **Model :** `php artisan make : model modelName`
> Note: a model name should be singular, and a table name should be plural.
- Ticket Model
```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
class Ticket extends Model
{
//
}
```
- Tells tickets belongs to users who creating the tickets
```php
public function user()
{
    return $this->belongsTo('App\User');
}
```
- Use this model to access any tickets’ data, such as: title, content, etc.
```php
public function getTitle()
{
    return $this->title
}
```
> It automatically finds and connects our models with our database tables if youname them correctly (Ticket model and tickets table, in this case).For some reasons, if you want to use a different name, you can let Eloquent know that by defininga table property like this:
> `protected $table = 'yourCustomTableName';`