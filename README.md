# How to deploy Laravel applications on shared hosting
[![API Documentation](http://img.shields.io/badge/en-English-brightgreen.svg)](README.md)

The simple guide to deploy Laravel and Lumen application on shared hosting.

## Requirements

Before trying to deploy a Laravel application on a shared hosting, you need to make sure that the hosting services provide a fit [requirement to Laravel](https://laravel.com/docs/5.2#server-requirements). Basically, following items are required for Laravel 8.x:

* PHP >= 7.4
* PDO PHP Extension
* Mbstring PHP Extension

Well, it also depends on the Laravel version you want to try to install, checkout the appropriate version of [Laravel server requirements documentation](https://laravel.com/docs/master).

Besides PHP and those required extensions, you also need these utilities to make the deployment much easier.

* [Git](https://git-scm.com/)
* [Composer](https://getcomposer.org/)

**Next, if you are cloning a privately-hosted remote repository, you need to have the SSH access permission for your hosting account; otherwise, you are good if it is a public repo.**

## Set up access to private repositories
To set up access to private repositories, perform the following steps:

### Generate an SSH key
If you have not already configured one, run the following command to generate an SSH key:

```ssh-keygen -t rsa -b 4096 -C "username@example.foo"```

In this example, username represents the cPanel account username and example.foo represents your domain name.

After you run this command, the system will prompt you to enter a passphrase. Do not enter a passphrase, and press Enter to continue.

### Confirm that you generated the SSH key correctly
To confirm that the key exists and is in the correct location, run the following command:

```cat ~/.ssh/id_rsa.pub```

The output should resemble the following example, where AAAAB3Nza... represents a valid SSH key:

```ssh-rsa AAAAB3Nza...```

### Register your SSH key with the private repository host
Note:
> For information about how to register your SSH key with another private repository host, consult that host’s website or documentation. Some repository hosts, such as Bitbucket, do not allow you to configure write access for your access keys.
To register an SSH key with GitHub, perform the following steps:

1. Log in to your GitHub account.

2. Navigate to your private repository.

3. In the top right corner of the page, click *Settings*. A new page will appear.

4. In the left side menu, click *Deploy keys*. A new page will appear.

5. In the top right corner of the page, click *Add deploy key*. A new page will appear.

6. In the *Title* text box, enter a display name for the key. I personally use the domain name.

7. In the *Key* text box, paste the entire SSH key.

8. If you want to push code from your cPanel account to your GitHub account, select the *Allow write* access checkbox.

Note:
> If you do not select this checkbox, you can only deploy changes from your GitHub repository to the cPanel-hosted repository.

9. Click *Add* key.

### Test the SSH key
To test your SSH key, run the following command, where `example.foo` represents the private repository’s host:

```ssh -T git@example.foo```

I guess that's enough for SSH. Please refer to below sections to learn more about deployment.

## Clone the repository

Let's get started by understanding how we should organize the Laravel application structure. At first, you will have this or similar directories and files in your account,

```bash
.bash_history
.bash_logout
.bash_profile
.bashrc
.cache
.cpanel
.htpasswds
logs
mail
public_ftp
public_html
.ssh
tmp
etc
www -> public_html
...
```

For the main account which is tied with the main domain, the front-end code should stay in `public_html` or `www`. Since, we don't want to expose *Laravel things* (such as, .env, ...) to the outside world, we will hide them.
There are two methods of cloning the repository. You can either clone through the terminal or use a wizard that comes with cpanel. Most people are not aware about the second method.

### (a) Clone through the Terminal
Create a new directory to store all the code, name it `repositories` or whatever you want to.

```bash
$ mkdir repositories
$ cd repositories
```

Alright, from here, just issue a git command to grab the code,

```bash
$ git clone http://[GIT_SERVER]/awesome-app.git
$ cd awesome-app
```

### (a) Clone through Git™ Version Control Wizard
Search `git` in your cPanel home.
Click the *Create* button located on the top right.
Enter the clone URL for the remote repository in the respective text field.
The rest of the fields should now be autofilled for you automatically. You can change the naming as you desire.
Now click *Create* and the wizard will start cloning your repository if SSH Setup was done correctly.

Next step is to make the `awesome-app/public` directory to map with `www` directory, symbol link is a great help for this, but we need to backup `public` directory first.

```bash
$ mv public public_bak
$ ln -s ~/www public
$ cp -a public_bak/* public/
$ cp public_bak/.htaccess public/
```

Because we created the symbol link from `www` directory to make it become the *virtual* `public` in project, so we have to update the `~/www/index.php` in order to replace paths with the new ones:

```diff
- require __DIR__.’/../bootstrap/autoload.php’;
+ require __DIR__.'/../repositories/awesome-app/bootstrap/autoload.php';

- $app = require_once __DIR__.’/../bootstrap/app.php’;
+ $app = require_once __DIR__.'/../repositories/awesome-app/bootstrap/app.php';
```

The updated file should be:

```php
require __DIR__.'/../repositories/awesome-app/bootstrap/autoload.php';

$app = require_once __DIR__.'/../repositories/awesome-app/bootstrap/app.php';
```

The hard part is done, the rest is to do some basic Laravel setup. Allow write permission to `storage` directory is important,

```bash
$ chmod -R o+w storage
```

**Edit your `.env` for proper configuration. Don't miss this!**

Lastly, update the required packages for Laravel project using **composer** and add some necessary caches,

```bash
$ php composer install
$ php composer dumpautoload -o
$ php artisan config:cache
$ php artisan route:cache
```

**Congratulation! You've successfully set up a Laravel application on a shared hosting service.**

## FAQs

> **1. How to acquire SSH access permission for my account?**

Just contact your hosting support, they will need to confirm your identity and will permit your SSH access in no time.

> **2. Where is git? I can't find it.**

`git` is often put under this place in CPanel hosting services, `/usr/local/cpanel/3rdparty/bin/git`. So you need to provide full path to `git` if you want to issue a `git` command; or, you can also create an alias for convenient use,

```bash
alias git="/usr/local/cpanel/3rdparty/bin/git"
```

> **3. How to get composer?**

You can use FTP or SCP command to upload `composer.phar` to the host after downloading it locally. Or use `wget` and `curl` to get the file directly on host,

```bash
$ wget https://getcomposer.org/composer.phar

or

$ curl -sS https://getcomposer.org/installer | php — –filename=composer
```

> **4. Does this work with Lumen?**

Well, Laravel and Lumen are like twins, so it applies the same with Lumen.

> **5. I try to run `composer` but it shows nothing. What is the problem?**

You need to provide a proper PHP configuration to run `composer`, which means, you cannot execute `composer` directly on some hosting service providers. So to execute `composer`, you will need to issue this command,

```bash
$ php -c php.ini composer [COMMAND]
```

> **6. Where can I get the `php.ini` to load for `composer`?**

You can copy the default PHP configuration file `php.ini`, which is often at `/usr/local/lib/php.ini`, or find it by this command,

```bash
$ php -i | grep "php.ini"
```

## List of service providers tested and worked

The following shared hosting service providers have been tested and worked perfectly 100%.

* [NameCheap](https://www.namecheap.com/)
* [JustHost](https://www.justhost.com/)
* [bluehost](https://www.bluehost.com/)
* [GoDaddy](https://godaddy.com/)
* [HostGator](http://www.hostgator.com/)
* [GeekStorage](https://www.geekstorage.com/)
* [Site5](https://www.site5.com/)

Works on [GeekStorage](https://www.geekstorage.com/) shared plan but I had to enable PHP 7.4 via `.htaccess`

```
AddHandler application/x-httpd-php74 .php
```

If you found any hosting providers that works, please tell me, I will update the list for others to know about them, too.

## Still trouble?

If you still fail to deploy Laravel applications after following all above steps. Provide me [your issue](https://github.com/michaelgatuma/laravel-deploy-on-shared-hosting/issues) in details, I will help you out.

## Contribution Guide

Free free to fork the project and submit [a pull request](https://github.com/michaelgatuma/laravel-deploy-on-shared-hosting/pulls).
