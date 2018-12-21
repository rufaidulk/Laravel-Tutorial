# Laravel - Multi Auth : 4 - User roles (ACL)
> Note: ACL stands for Access Control List
- So first creat the model and migration for roles
```php
php artisan make:model Role -m
php artisan make:model RoleAdmin -m
```
- Update the schema of the **roles** migration as follows:
```php
public function up()
    {
        Schema::create('roles', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->timestamps();
        });
    }
```
- Update the schema of the **role_admins** migration as follows:
```php
public function up()
    {
        Schema::create('role_admins', function (Blueprint $table) {
            $table->increments('id');
            $table->integer('role_id')->unsigned(();
            $table->integer('admin_id')->unsigned(();
            $table->timestamps();
        });
    }
```
-     Now run the **`php artisan migrate`**
---
- Now manually insert a new admin to admin table, i.e via phpMyAdmin
- Also add 2 roles for **role** table 
1. admin // id will be 1
1. editor // id will be 2
- insert data as follows in **role_admins** table
```php
role_id     admin_id
1              1     //means first user in admin table is admin
2              2     //means second user in admin table is editor
```
- Now lets describe the relationships between role and admin, go to **Admin** model and update as follows:
```php

```
