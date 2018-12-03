# Laravel - Queue
Queues are used to create a delay or defer a time-consuming task, e.g: sending emails etc. Deferring these type of time-consuming task will speed up the web requests to the application. For example, in case of sending email using queues, when the user requests to send the mail the requests will be processed immediately. but the sending email will be done in the background with a delay.    

## **Configure .env file to send mails**
- **open the .env file and find the code snippet shown below.**
```php
MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```
> **smtp** stands for **simple mail transfer protocol,** it is an internet standard for sending emails
- **Add mailtrap credentials to the .env file. go to [mailtrap.io](http://mailtrap.io) and create an account, then open demo inbox where you can see the smtp credentials.**
- **Copy the Host, Username, Password and repalce it in the .env file.**
```php
MAIL_DRIVER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=<your-username>
MAIL_PASSWORD=<your-password>
MAIL_ENCRYPTION=tls
```
> **tls** stands for **transport layer security.** it is a protocol used for the encrypt a communication channel between two computers over the internet.
- **In Laravel, each mail sent by the application is represented as "mailable" class. all mailable class are in "App/Mail" directory. To create an mailable class run this command:**
```php
php artisan make:mail <mailable-class-name> 
//for e.g: 
php artisan make:mail SendInfoEmail
```
- **open the *SendInfoEmail.php* in "App/Mail" directory. then update the *build method* for returning a view. here view is the content of the mail.**
```php
public function build()
{
    return $this->view('view.name');// e.g: view('welcome') for welcome.blade.php
}
```
- **for sending email we are using  *web.php* add the following code**
```php
use App\Mail\SendInfoEmail;
use Illuminate\Support\Facades\Mail;

Route::get('sendmail', function(){
    Mail::to('mail@example.com')->send(new SendInfoEmail());
    return "email sent successfully";
}
```
- **then go to *laravel.test/sendmail*  email will sent you can see the return message.**
- **for decreasing the request processing we can use Queue.**
---
## **Send Email using Queues**
-  **Here we are using Database queue driver. so a database table is needed to hold the jobs.**
- **Run the following command to generate the jobs table migration:**
```php
php artisan queue:table
```
- **migration will create and a file name *create_jobs_table.php*   is created "database/migrations" folder.**
- **run the following command to create migration table :**
```php
php artisan migrate
```
- ***jobs* table is created in the database.**
- **Now create the jobs by running the following command:**
```php
php artisan make:job <job-name> 
//for e.g: 
php artisan make:job SendInfoMailJob
```
- ***SendInfoMailJob.php*  is created in "App/Jobs" directory.**
- **write the logic for the job we want to do in the *handle method*  in  *SendInfoMailJob.php* file**
```php
public function handle()
{
    //our job is to send mail
    Mail::to('mail@example.com')->send(new SendInfoMail());
}
```
- **next  we have to trigger the job or dispatch the job. for that add following code snippet:**
```php
SendInfoMailJob::dispatch(); // here SendInfoMailJob is the name of the job we created
```
> you can dispatch the job in controller also, here we are dispatching it in **web.php**
- **So update your *web.php*  file as follows:**
```php
use App\Jobs\SendInfoMailJob;

Route::get('sendmail', function(){
    SendInfoMailJob::dispatch();
    return "email sent successfully";
}
```
- **Still takes time for loading, so we have to send the email background, i.e delayed dispatching.**
- **that means page loads immediately and email will send in background with a small delay.**
- **for that update *web.php*  as follows:**
```php
use App\Jobs\SendInfoMailJob;

Route::get('sendmail', function(){
     SendInfoMailJob::dispatch()
                ->delay(now()->addSeconds(5));
                
    return "email sent successfully";
    
}
```
- **Now open *config/queue.php*   , you can see code snippet as follows:**
```php
'default' => env('QUEUE_CONNECTION', 'sync'),
```
here the default configuration is ****sync**** , i.e don't delay anything. so we have to replace sync  with **database.**  in order to do that open **.env file** and update as follows:
```php
QUEUE_CONNECTION=database
```
> Note: restart your server since **.env** is updated.
- **Now for processing the job we need to run the queue work. run this command:**
```php
php artisan queue:work
```
- **You can see the status of Queue Work processing in the terminal. Now the email will be sent after 5 second when request receieved.**
###             **Finished!**