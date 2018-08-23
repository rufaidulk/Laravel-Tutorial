# Laravel Note - Migrations
- Create a new migration file 
        `php artisan make : migration create_tablename_table --create = tablename`
        
        **--create** option automatically generates the codes to create the tickets table for
you.
1. **up** method: you use this method to add new tables, column to the database.
1. **down** method: well, you might have guessed already, this method is used to reverse what you’ve created.
```php
public function up()
{
    Schema::create('tickets', function (Blueprint $table) {
        $table->increments('id');
        $table->timestamps();
    });
}
```
- **Schema** is a class that we can use to define and manipulate tables. We use **Schema::create** method to create the tickets table
- **Schema::create** method has two parameters. The first one is the name of the table.
- The second one is a **Closure**. The **Closure** has one parameter: **$table**. You can name the parameter whatever you like.
- Use the **$table** parameter to create database columns, such as id, name, date, etc
- **increments(‘id’)** command is used to create id column and defines it to be an auto-increment primary key field in the tickets table.
- **timestamps** is a special method of Laravel. It creates updated_at and created_at column. Laravel uses these columns to know when a row is created or changed.
```php
<?php
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Database\Migrations\Migration;
class CreateTicketsTable extends Migration
{
/**
* Run the migrations.
*
* @return void
*/
    public function up()
    {
        Schema::create('tickets', function (Blueprint $table) {
            $table->increments('id');
            $table->string('title', 255);
            $table->text('content');
            $table->string('slug')->nullable();
            $table->tinyInteger('status')->default(1);
            $table->integer('user_id');
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
        Schema::drop('tickets');
    }
}
```
- to create the tickets table and its columns: `php artisan migrate`