> Прежде чем начинать убедитесь, что пройдены предыдущие этапы:  
> https://github.com/Webdjan/ubuntu/blob/master/readmy-steps.md

# Этап 4

### Django Postgres Gunicorn

> Обновляем индекс пакетов, устанавливаем новые

```text
sudo apt update
sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib
```

### Создаем базу данных

```text
sudo -u postgres psql
CREATE DATABASE myproject;
CREATE USER neytonuser WITH PASSWORD 'MyPassWord';
ALTER ROLE neytonuser SET client_encoding TO 'utf8';
ALTER ROLE neytonuser SET default_transaction_isolation TO 'read committed';
ALTER ROLE neytonuser SET timezone TO 'UTC';
GRANT ALL PRIVILEGES ON DATABASE myproject TO neytonuser;
\q
```

### Без комментариев (виртуальное окружение вкладываю внутрь корня будущего проекта, если нужно на уровень выше, просто поправьте пути)

```text
sudo -H pip3 install --upgrade pip
sudo apt install python3-venv
mkdir ~/myprojectdir
cd ~/myprojectdir
virtualenv myprojectenv
```
 ### ВХОДИМ В ВИРТУАЛЬНОЕ ОКРУЖЕНИЕ

```text
source myprojectenv/bin/activate
pip install django gunicorn psycopg2-binary
django-admin.py startproject myproject ~/myprojectdir
sudo nano ~/myprojectdir/myproject/settings.py
```

```python
. . .
ALLOWED_HOSTS = ['mydomen.ru', 'www.mydomen.ru', 'localhost']
. . . . .
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'myproject',
        'USER': 'neytonuser',
        'PASSWORD': 'MyPassWord',
        'HOST': 'localhost',
        'PORT': '',
    }
}
. . . . .
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
. . . . .

```

```text
~/myprojectdir/manage.py makemigrations
~/myprojectdir/manage.py migrate
~/myprojectdir/manage.py createsuperuser
~/myprojectdir/collectstatic
```

### ВЫХОДИМ ИЗ ВИРТУАЛЬНОГО ОКРУЖЕНИЯ

> sudo nano /etc/systemd/system/gunicorn.socket

```text
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

> sudo nano /etc/systemd/system/gunicorn.service

```text
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=neyton
Group=www-data
WorkingDirectory=/home/neyton/myprojectdir
ExecStart=/home/neyton/myprojectdir/myprojectenv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          myproject.wsgi:application

[Install]
WantedBy=multi-user.target
```

```text
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket

sudo systemctl status gunicorn.socket
file /run/gunicorn.sock

sudo systemctl status gunicorn
curl --unix-socket /run/gunicorn.sock localhost
sudo systemctl status gunicorn
```

### Конфиг nginx нашего серверного блока

> sudo nano /etc/nginx/sites-available/mydomen.ru  
> ВНИМАТЕЛЬНО СМОТРЕТЬ И ПРАВИТЬ ДОМЕННОЕ ИМЯ НА СВОЕ  
> заголовки add_header  - тут будьте еще внимательнее, прочитайте о них в доках  
> они настраиваются индивидуально для каждого сайта (можно их пока удалить)  
> если кратко - то работают по принципу: что не разрешено, то запрещено

```text
server {
    listen [::]:80;
    listen 80;
    server_name mydomen.ru www.mydomen.ru;
    return 301 https://mydomen.ru$request_uri;
}

server {
    listen [::]:443 ssl http2;
    listen 443 ssl http2;
    server_name www.mydomen.ru;
    ssl_certificate /etc/letsencrypt/live/mydomen.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/mydomen.ru/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
    root /home/neyton/myprojectdir;
    return 301 https://mydomen.ru$request_uri;
}

server {
    listen [::]:443 ssl http2;
    listen 443 ssl http2;
    server_name mydomen.ru;
    ssl_certificate /etc/letsencrypt/live/mydomen.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/mydomen.ru/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

    location = /favicon.ico {
        access_log off;
        log_not_found off;
    }

    location /static/ {
        root /home/neyton/myprojectdir;
        expires 30d;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
        root /home/neyton/myprojectdir;
        expires 30d;
    }
 

    location / {

        proxy_connect_timeout 5;
        proxy_send_timeout 8;
        proxy_read_timeout 8;
        proxy_temp_file_write_size 64k;
        proxy_buffer_size 4k;
        proxy_buffers 32 16k;
        proxy_busy_buffers_size 32k;
        proxy_cache_valid 1h;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_headers_hash_max_size 512;
        proxy_headers_hash_bucket_size 128;


        limit_conn perip 5;	
        proxy_redirect off;
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;

        add_header Content-Security-Policy "script-src 'self'; object-src 'self'";
        add_header X-Xss-Protection "1; mode=block" always;
        add_header x-frame-options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header Access-Control-Allow-Origin "https://mydomen.ru";
        add_header Referrer-Policy "strict-origin-when-cross-origin";

        add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";

        add_header Feature-Policy "accelerometer 'none';ambient-light-sensor 'none';autoplay 'none';camera 'none';encrypted-media 'none';fullscreen 'self';geolocation 'self';gyroscope 'none';magnetometer 'none';microphone 'none';midi 'none';payment 'self';picture-in-picture 'none';speaker 'self';sync-xhr 'none';usb 'none';vibrate 'none';vr 'none';";   
    }
}
```



### Заменяем все что в дефолте:


> sudo nano /etc/nginx/nginx.conf

```text
user www-data;
worker_processes auto;
worker_rlimit_nofile 3072;
pcre_jit on;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 1024;
}

http {
 sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	reset_timedout_connection on;
	keepalive_timeout 40;
	keepalive_requests 80;
	send_timeout 2;

	client_body_timeout 10;
	client_max_body_size 1m;
        types_hash_max_size 2048;
	server_tokens off;
	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	server_names_hash_bucket_size 64;
        client_body_buffer_size 16k;
        client_header_buffer_size 1k;
        large_client_header_buffers 2 1k;
        limit_conn_zone $binary_remote_addr zone=perip:6m;
	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;
	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;


	gzip on;
	gzip_static on;
	gzip_vary on;
	gzip_proxied any;
	gzip_comp_level 6;
	gzip_min_length 1000;
	gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss    text/javascript;

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```

```text
sudo nginx -t
sudo service nginx restart         (перезапуск nginx)
sudo systemctl restart gunicorn    (перезапуск гуникорн)

```

```text
Все работает, проверил, но не знаю все ли правильно сделал


Проверяем всё с помощью сервисов:
https://securityheaders.com/
https://www.webpagetest.org/
```

