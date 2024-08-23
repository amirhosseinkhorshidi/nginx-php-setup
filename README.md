# راهنمای نصب و پیکربندی وب‌سرور Nginx با PHP و MySQL

این راهنما به شما کمک می‌کند تا یک وب‌سرور با Nginx، PHP، و MySQL را بر روی یک سرور Ubuntu نصب و پیکربندی کنید. این دستورالعمل‌ها شامل نصب نرم‌افزارهای مورد نیاز، پیکربندی Nginx برای پشتیبانی از PHP، ایجاد یک پایگاه داده MySQL، و نصب ماژول‌های PHP است. همچنین، نحوه رفع برخی از خطاهای رایج و تنظیم کرون‌جاب را نیز پوشش می‌دهد.

## ۱. به‌روزرسانی سیستم

قبل از هر کاری، باید سیستم خود را به‌روزرسانی کنید تا مطمئن شوید که همه بسته‌ها و نرم‌افزارهای موجود به آخرین نسخه‌ها به‌روزرسانی شده‌اند:

```bash
sudo apt update
sudo apt upgrade -y
```

این دستورات لیست بسته‌ها را به‌روزرسانی کرده و سپس تمام بسته‌های نصب‌شده را به آخرین نسخه‌ها به‌روزرسانی می‌کند.

## ۲. نصب وب‌سرور Nginx

برای نصب وب‌سرور Nginx از دستور زیر استفاده کنید:

```bash
sudo apt install nginx -y
```

این دستور Nginx را روی سرور شما نصب می‌کند و به‌صورت خودکار سرویس Nginx را شروع می‌کند.

## ۳. نصب MySQL

### نصب MySQL

برای نصب MySQL و انجام تنظیمات اولیه امنیتی، دستورات زیر را اجرا کنید:

```bash
sudo apt install mysql-server -y
sudo mysql_secure_installation
```

در حین اجرای دستور `mysql_secure_installation`، به ترتیب به سؤالات پاسخ دهید:
- اولین پرسش را با `N` پاسخ دهید. (استفاده از پلاگین اعتبارسنجی رمز عبور)
- سایر سؤالات را با `Y` پاسخ دهید.

### ایجاد کاربر و تنظیمات امنیتی

برای ایجاد یک کاربر جدید MySQL و تنظیم رمز عبور، ابتدا وارد کنسول MySQL شوید:

```bash
sudo mysql
```

سپس دستورات زیر را اجرا کنید تا یک کاربر جدید با دسترسی‌های کامل ایجاد شود:

```sql
CREATE USER 'Your_USER'@'localhost' IDENTIFIED BY 'Your_PASS';
GRANT ALL PRIVILEGES ON *.* TO 'Your_USER'@'localhost' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

**توجه:** `Your_USER` را با نام کاربری دلخواه و `Your_PASS` را با رمز عبور دلخواه خود جایگزین کنید.

## ۴. نصب PHP و ماژول‌های مرتبط

### نصب PHP

برای نصب PHP و ماژول‌های مرتبط مورد نیاز برای اجرای PHP در Nginx، از دستورات زیر استفاده کنید:

```bash
sudo apt install php-fpm php-mysql -y
```

### نصب ماژول‌های اضافی PHP

اگر به ماژول‌های بیشتری نیاز دارید، از دستور زیر استفاده کنید: (پیشنهادی)

```bash
sudo apt install php8.1-amqp php8.1-apcu php8.1-bcmath php8.1-calendar php8.1-ctype php8.1-curl php8.1-intl php8.1-enchant php8.1-exif php8.1-ftp php8.1-gd php8.1-gmp php8.1-igbinary php8.1-imagick php8.1-imap php8.1-intl php8.1-ldap php8.1-mbstring php8.1-memcache php8.1-memcached php8.1-mongodb php8.1-msgpack php8.1-mysql php8.1-oauth php8.1-opcache php8.1-raphf php8.1-redis php8.1-sqlite3 php8.1-tidy php8.1-uuid php8.1-xdebug php8.1-xml php8.1-xmlrpc php8.1-yaml -y
```

**نکته:** تنها ماژول‌هایی را که برای پروژه شما ضروری هستند نصب کنید. نصب ماژول‌های اضافی ممکن است باعث استفاده بیش از حد از منابع سیستم شود.

## ۵. پیکربندی Nginx برای پشتیبانی از PHP

### تنظیمات SSL

ابتدا فایل‌های SSL (`fullchain.pem` و `privkey.pem`) را به یک مسیر امن روی سرور خود کپی کنید. برای این کار دستورات زیر را اجرا کنید:

```bash
sudo mkdir -p /etc/nginx/ssl
sudo cp /path/to/fullchain.pem /etc/nginx/ssl/
sudo cp /path/to/privkey.pem /etc/nginx/ssl/
sudo chmod 600 /etc/nginx/ssl/privkey.pem
sudo chmod 644 /etc/nginx/ssl/fullchain.pem
```

### ویرایش فایل پیکربندی Nginx

فایل پیکربندی Nginx را برای فعال‌سازی HTTPS و پشتیبانی از PHP ویرایش کنید:

```bash
sudo nano /etc/nginx/sites-available/default
```

فایل را به صورت زیر پیکربندی کنید:

```nginx
server {
    listen 80;
    server_name DOMAIN.COM www.DOMAIN.COM;
    return 301 https://$host$request_uri;  # ریدایرکت از HTTP به HTTPS
}

server {
    listen 443 ssl;
    server_name DOMAIN.COM www.DOMAIN.COM;

    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    root /var/www/html;
    index index.php index.html index.htm index.nginx-debian.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }

    location /phpmyadmin {
        alias /usr/share/phpmyadmin/;
        index index.php;
        try_files $uri $uri/ =404;

        location ~ ^/phpmyadmin/(doc|sql|setup)/ {
            deny all;
        }

        location ~ /phpmyadmin/(.+\.php)$ {
            fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
            include snippets/fastcgi-php.conf;
            fastcgi_param SCRIPT_FILENAME $request_filename;
        }
    }
}
```

**نکته:** مسیر `fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;` ممکن است بسته به نسخه PHP نصب‌شده روی سیستم شما متفاوت باشد. حتماً بررسی کنید که این مسیر با نسخه PHP نصب‌شده در سیستم شما مطابقت داشته باشد.

### بررسی و راه‌اندازی مجدد Nginx

پس از انجام تغییرات، پیکربندی Nginx را بررسی و سرویس را راه‌اندازی مجدد کنید:

```bash
sudo nginx -t
sudo systemctl restart nginx
```

### ۶. ایجاد پایگاه داده و دسترسی‌ها

ابتدا phpMyAdmin را با دستور زیر نصب کنید:

```bash
sudo apt install phpmyadmin -y
```

برای ایجاد یک پایگاه داده در MySQL و اختصاص دسترسی‌های لازم به یک کاربر، ابتدا وارد MySQL شوید:

```bash
sudo mysql
```

سپس دستورات زیر را اجرا کنید:

```sql
CREATE DATABASE mydatabase;
GRANT ALL PRIVILEGES ON mydatabase.* TO 'Your_USER'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

اکنون می‌توانید با رفتن به آدرس `yourdomain.com/phpmyadmin` به phpMyAdmin دسترسی داشته باشید.

## نکات اضافی

### ۱. رفع خطاها

#### نصب اکستنشن Soap

برای رفع خطای `Uncaught Error: Class "SoapClient" not found`، اکستنشن soap را نصب و فعال کنید:

```bash
sudo apt install php-soap -y
sudo systemctl restart php8.1-fpm
sudo systemctl restart nginx
```

**نکته:** مطمئن شوید که نسخه اکستنشن نصب‌شده با نسخه PHP شما مطابقت دارد. به عنوان مثال، اگر از PHP 8.1 استفاده می‌کنید، باید اکستنشن `php8.1-soap` نصب شود.

### ۲. تنظیم دسترسی‌ها

برای تنظیم دسترسی‌های صحیح به فایل‌ها و دایرکتوری‌ها، دستورات زیر را اجرا کنید:

```bash
sudo chown -R www-data:www-data /var/www/html/
sudo chmod -R 755 /var/www/html/
```

### ۳. تغییر گروه مالک و اعمال بیت Setgid

دستورات زیر به ترتیب برای تغییر گروه مالک دایرکتوری و اعمال بیت Setgid روی آن استفاده می‌شوند:

```bash
sudo chgrp -R www-data /var/www/html/
sudo chmod g+s /var/www/html/
```

- **دستور اول (`chgrp`)**: گروه مالک دایرکتوری و تمامی فایل‌های داخل آن را به `www-data` تغییر می‌دهد.
- **دستور دوم (`chmod g+s`)**: بیت Setgid را روی دایرکتوری تنظیم می‌کند تا فایل‌هایی که در آینده در این دایرکتوری ایجاد می‌شوند، به طور خودکار گروه مالک دایرکتوری را به ارث ببرند.

### ۴. تنظیم کرون‌جاب

برای اجرای یک اسکریپت PHP به صورت منظم با استفاده از کرون‌جاب، دستور زیر را در کرون‌جاب وارد کنید:

```bash
crontab -e
 * * * /usr/bin/php /var/www/html/test/test.php
```

این دستور اسکریپت `test.php` را هر دقیقه اجرا می‌کند.

**نکته:** اجرای کرون‌جاب هر دقیقه ممکن است منابع سیستم را بیش از حد مصرف کند. در صورت نیاز به اجرای کمتر مکرر، تنظیمات مناسب‌تر را انتخاب کنید.

### ۵. بررسی لاگ‌ها

برای بررسی مشکلات احتمالی، لاگ‌های Nginx و PHP-FPM را چک کنید:

```bash
sudo tail -f /var/log/nginx/error.log
sudo tail -f /var/log/php8.1-fpm.log
```

### ۶. مدیریت آپلود فایل‌های حجیم

#### تغییر تنظیمات PHP

##### ویرایش فایل `php.ini`

فایل `php.ini` را ویرایش کنید:

```bash
sudo nano /etc/php/8.1/fpm/php.ini
```

مقادیر زیر را تغییر دهید:

```ini
upload_max_filesize = 10M
post_max_size = 12M
```

##### ریستارت PHP-FPM

برای اعمال تغییرات، PHP-FPM را ریستارت کنید:

```bash
sudo systemctl restart php8.1-fpm
```

#### تغییر تنظیمات Nginx

##### ویرایش فایل پیکربندی Nginx

فایل پیکربندی Nginx را ویرایش کنید:

```bash
sudo nano /etc/nginx/nginx.conf
```

یا فایل تنظیمات سایت:

```bash
sudo nano /etc/nginx/sites-available/default
```

خط زیر را اضافه یا تغییر دهید:

```nginx
client_max_body_size 10M;
```

##### ریستارت Nginx

برای اعمال تغییرات، Nginx را ریستارت کنید:

```bash
sudo systemctl restart nginx
```

### ۷. نصب و راه‌اندازی UFW (فایروال)

برای افزایش امنیت سرور، توصیه می‌شود از فایروال UFW استفاده کنید. ابتدا UFW را نصب و سپس پورت‌های مورد نیاز را باز کنید:

```bash
sudo apt install ufw -y
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw allow پورت دلخواه
sudo ufw enable
```

- **پورت ۲۲** برای دسترسی SSH
- **پورت ۸۰** برای ترافیک HTTP
- **پورت ۴۴۳** برای ترافیک HTTPS

پس از اعمال این تنظیمات، UFW را فعال کنید تا سرور شما در برابر ترافیک‌های ناخواسته محافظت شود.

**نکته:** اگر برنامه‌ای دیگری دارید که به پورت‌های دیگری نیاز دارد، باید آن پورت‌ها را نیز باز کنید. به عنوان مثال، اگر برنامه‌ای روی پورت 8080 اجرا می‌شود، باید آن را با دستور `sudo ufw allow 8080` باز کنید.
