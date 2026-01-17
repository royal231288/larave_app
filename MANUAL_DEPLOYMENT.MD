Laravel Deployment Guide (EC2 / Ubuntu / Nginx / PHP-FPM)
1️⃣ Server Setup
Update and install dependencies:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt update
sudo apt install -y software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update

sudo apt install -y php8.4 php8.4-fpm php8.4-mbstring php8.4-xml php8.4-bcmath php8.4-mysql php8.4-curl unzip git curl nginx mysql-client

```

Install composer 
```bash 
curl -sS https://getcomposer.org/installer | php
sudo mv composer.phar /usr/local/bin/composer
sudo chmod +x /usr/local/bin/composer
```

Install mysql 

```bash 
sudo apt update
sudo apt install -y mysql-server
```
Set up root user password (Optinal)
```bash
sudo mysql
# In MySQL shell
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YourStrongRootPassword';
FLUSH PRIVILEGES;
EXIT;
```
Create user for the database and set password for the user 
```bash
CREATE DATABASE laravel_db;
CREATE USER 'laravel_user'@'localhost' IDENTIFIED BY 'strong_password';
CREATE USER 'laravel_user'@'127.0.0.1' IDENTIFIED BY 'strong_password';
GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'localhost';
GRANT ALL PRIVILEGES ON laravel_db.* TO 'laravel_user'@'127.0.0.1';
FLUSH PRIVILEGES;
EXIT;
```
3️⃣ Project Setup

Assuming your project is in /home/ubuntu/laravel_app:
```bash
cd /home/ubuntu
git clone <your-repo-url> laravel_app
cd laravel_app
composer install --no-dev --optimize-autoloader
cp .env.example .env
php artisan key:generate

```
Update necessay db credential on .env
```bash
nano .env
```
set credential 
```bash
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel_db
DB_USERNAME=laravel_user
DB_PASSWORD=strong_password
```

Giving necessary permission to www-data user so that nginx can read write and execute files

Create a common group for deployment user(ubuntu) & web server(www-data):
```bash
sudo groupadd laravel       # only if it doesn’t exist
sudo usermod -aG laravel ubuntu
sudo usermod -aG laravel www-data
```
Set ownership and group permission
```bash
sudo chown -R ubuntu:laravel .                     # Code owned by deployment user
sudo chown -R ubuntu:laravel storage bootstrap/cache  # Writable dirs owned by deployment user, group laravel
```
Set permission
```bash
sudo chmod 755 /home          # /home should be globally readable & traversable
sudo chmod 755 /home/ubuntu   # Keep private for ubuntu & group, www-data in group 'laravel' can traverse

# Directories: owner+group read/write/execute
sudo find storage bootstrap/cache -type d -exec chmod 775 {} \;

# Files: owner+group read/write
sudo find storage bootstrap/cache -type f -exec chmod 664 {} \;

# Make artisan executable
sudo chmod +x artisan

# Make new files inherit group automatically
sudo find storage bootstrap/cache -type d -exec chmod g+s {} \;
```
Now you can run the necessary laravel provided command without any permission error
-- Run migrations
```bash
php artisan migrate --force

```
Clear Cache 
```bash 
php artisan cache:clear
php artisan config:clear
php artisan route:clear
php artisan view:clear
```
Rebuild the cache
```bash
php artisan config:cache
php artisan route:cache
php artisan view:cache
```
All commands can be run as ubuntu (deployment user) thanks to group permissions.

Now create symbolic link to the /var/www/laravel_app from the laravel_app/public 
here laravel_app actually will act like public directory
```bash 
sudo ln -s /home/ubuntu/laravel_app/public /var/www/laravel_app
```
Now update nginx configuration
```bash
sudo nano /etc/nginx/sites-available/laravel_app
```
copy given configuration and added to the laravel_app
fastcgi_pass unix:/run/php/php8.4-fpm.sock; // for the used php version
close the file cll+O+X
```bash 
server {
    listen 80;
    server_name _;

    root /var/www/laravel_app;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.4-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}

```

Now enable the sites and reloading nginx 
```bash 
sudo ln -s /etc/nginx/sites-available/laravel_app /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

```
Remove default configuration
```bash 
sudo rm /etc/nginx/sites-enabled/default
```
checking permission 
```bash
ls -ld /home /home/ubuntu /home/ubuntu/laravel_app
```
expected output: 
```bash
drwxr-xr-x  3 root   root    4096 Jan 17 12:38 /home
drwxr-xr-x  7 ubuntu ubuntu  4096 Jan 17 13:32 /home/ubuntu
drwxr-xr-x 13 ubuntu laravel 4096 Jan 17 13:50 /home/ubuntu/laravel_app
```

