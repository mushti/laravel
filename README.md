# Laravel Wiki
Tips, tricks and how-tos for your Laravel projects.

## Authentication
Use `php artisan make:auth` command to implement authentication in your Laravel project. Visit the [offical documentation](https://laravel.com/docs/5.6/authentication) for more details on how to setup authentication.
### Custom Validation
There are two different ways to implement custom validation of the user credentials, depending on the scenario.
If you want to check if a user being authenticated has a `role` of `admin`, then you just need to override the `credentials()` method of the `AuthenticatesUsers` trait being used inside the `LoginController`.
```php
/**
 * Get the needed authorization credentials from the request.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return array
 */
protected function credentials(Request $request)
{
    return [
        $this->username() => $request->{$this->username()},
        'password' => $request->password,
        'role' => 'admin'
    ];
}
```
If you want to check if a user being authenticated has a `role` other than `customer`, then you need to override the `attemptLogin()` method of the `AuthenticatesUsers` trait being used inside the `LoginController`.
```php
/**
 * Attempt to log the user into the application.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return bool
 */
protected function attemptLogin(Request $request)
{
    if (
        $this->guard()->attempt(
            $this->credentials($request), $request->filled('remember')
        ) && $this->guard()->user()->type != 'customer'
    ) return true;
    
    $this->guard()->logout();
    return false;
}
```

## EC2 Deployment
```
apt-get update
apt-get upgrade
```
### Apache 2
```
apt-get install apache2
a2enmod rewrite
service apache2 restart
```
### MySQL
```
apt-get install mysql-server
mysql_secure_installation
```
When connecting to the MySQL database, if you get an authentication error, kindly reset your root password using the following commands:
```
mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YOUR_NEW_PASSWORD';
```
### PHP 7.2
```
add-apt-repository -y ppa:ondrej/php
apt-get update
apt-get install -y php7.2-fpm
apt-get install -y php7.2
apt-get -y install curl php-pear php7.2-mysql php7.2-dev php7.2-curl php7.2-json php7.2-mbstring php7.2-gd php7.2-intl php7.2-xml php7.2-imagick php7.2-redis php7.2-zip libapache2-mod-php 
systemctl restart apache2
```
### Git
```
apt-get install git
```
### Composer
```
cd ~
curl -sS https://getcomposer.org/installer -o composer-setup.php
php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
https://composer.github.io/pubkeys.html
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```
### Setup
```
cd /var/www
rm -r html
git clone https://gitlab.com/mushti/chalo.git
cd {project_folder}
composer install
nano .env
php artisan migrate
php artisan passport:install

chown -R ubuntu:www-data /var/www/{project_folder}
find /var/www/{project_folder} -type f -exec chmod 664 {} \;
find /var/www/{project_folder} -type d -exec chmod 775 {} \;
chgrp -R www-data storage bootstrap/cache
chmod -R ug+rwx storage bootstrap/cache

nano /etc/apache2/sites-enabled/000-default.conf
Change DocumentRoot /var/www/{project_folder}/public
    <Directory /var/www>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        allow from all
    </Directory>
systemctl restart apache2
```