# Laravel Deployment
Deployment of Laravel project on different types of servers.

## EC2 Deployment
Begin the deployment by updating the local package index to reflect the latest upstream changes.
```
apt-get update
apt-get upgrade
```
### Install Apache 2
Now install Apache server.
`apache2` package is available within Ubuntu's software repositories, so we will install it using `apt`.
```
apt-get install apache2
```
### Install MySQL
```
apt-get install mysql-server
mysql_secure_installation
```
When connecting to the MySQL database, if you get an authentication error, kindly reset your root password using the following commands:
```
mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YOUR_NEW_PASSWORD';
```
### Install PHP
To install `PHP` you first need to add `ppa:ondrej/php` to the `apt` repository.
```
add-apt-repository -y ppa:ondrej/php
apt-get update
```
Now use the following command to install `php-fpm` and `php`.
```
apt-get install -y php-fpm
apt-get install -y php
```
Now install some necessary php extensions and restart the server, using the following commands:
```
apt-get -y install curl php-pear php-mysql php-dev php-curl php-json php-mbstring php-gd php-intl php-xml php-imagick php-redis php-zip libapache2-mod-php
systemctl restart apache2
```
### Install Git
```
apt-get install git
```
### Install Composer
```
cd ~
curl -sS https://getcomposer.org/installer -o composer-setup.php
```
Visit [Composer's Pubkeys](https://composer.github.io/pubkeys.html) page to get the latest `SHA384` hash and replace it with `LATEST_SHA384_HASH` in the following command.
```
php -r "if (hash_file('SHA384', 'composer-setup.php') === 'LATEST_SHA384_HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```
### Setup
```
a2enmod rewrite
systemctl restart apache2
```
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
If you get the following error,
```
In CryptKey.php line 45:

Key path "file:///home/.../public_html/storage/oauth-private.key" does not exist or is not readable
```
Run the following commands to create `oauth-private.key` and `oauth-public.key` files.
```
openssl genrsa -out storage/oauth-private.key 4096
openssl rsa -in storage/oauth-private.key -pubout > storage/oauth-public.key
```