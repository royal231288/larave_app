Laravel Deployment Guide (EC2 / Ubuntu / Nginx / PHP-FPM)
1️⃣ Server Setup
Update and install dependencies:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y php8.3 php8.3-fpm php8.3-mbstring php8.3-xml php8.3-bcmath php8.3-mysql php8.3-curl unzip git curl composer nginx mysql-client

```bash


Use the PHP version that matches your Laravel project (8.3 in your case)

Include required extensions (mbstring, xml, bcmath, curl, mysql)

