# Laravel Notes - New Project
- Go to the Homestead directory via terminal :  `cd Homestead`
- Open Homestead.yaml file via nano editor : `nano Homestead.yaml`
```php
sites:
    - map: homestead.test
      to: /home/vagrant/code/Laravel/public
    - map: newAppName.test
      to: /home/vagrant/code/newAppName/public
    - map: phpmyadmin.test
      to: /home/vagrant/code/phpMyAdmin
      
```
      
> Note: **code** is your laravel project directory created in first laravel installation tutorial 
- Edit **/etc/hosts** file  to know the system about the new changes
- Run : `sudo nano /etc/hosts` in terminal. add following lines
```php
192.168.10.10  homestead.test
192.168.10.10  newAppName.test
```
- Run `vagrant provision` if vagrant is already running. otherwise run: `vagrant up`
> Note: check vagrant running or not using this command : `vagrant status`
- Now ssh to your virtual box. run: `vagrant ssh`
- Go to project directory : `cd code`
## Install new laravel app
- check the **laravel installer** up to date by running this command :
            `composer global require "laravel/installer"`
*will show output like this:*
```php
Changed current directory to /home/vagrant/.composer
Using version ^2.0 for laravel/installer
./composer.json has been updated
Loading composer repositories with package information
Updating dependencies (including require-dev)
Package operations: 0 installs, 1 update, 0 removals
  - Updating laravel/installer (v1.5.0 => v2.0.1): Loading from cache
Writing lock file
Generating autoload files
```
-  run : `laravel new newAppName`
*wil get an output like this*
```php
Discovered Package: fideloper/proxy
Discovered Package: laravel/tinker
Discovered Package: nesbot/carbon
Discovered Package: nunomaduro/collision
Package manifest generated successfully.
Application ready! Build something amazing.
```
- Now go to your browser and enter `newAppName.test` you can see the laravel default home page
- You can stop vagrant box by running : `vagrant halt`