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

### Без комментариев (за...ся уже, виртуальное окружение вкладываю внутрь корня будущего проекта, если нужно на уровень выше, просто поправьте пути)

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
ExecStart=/home/neyton/myprojectenv/bin/gunicorn \
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
> ВНИМАТЕЛЬНО СМОТРЕТЬ И ПРАВИТЬ ДОМЕННОЕ ИМЯ НА СВОЕ(я сам оху....л)

```text
server {
    listen [::]:80;
    listen 80;

    server_name mydomen.ru www.mydomen.ru;
    
    # redirect http to https www
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

    # redirect https non-www to https www
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
    }

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_headers_hash_max_size 512;
        proxy_headers_hash_bucket_size 128;
        proxy_redirect off;
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;

        add_header Content-Security-Policy "img-src * 'self' data: blob: https:; default-src 'self' https://*.googleapis.com https://*.googletagmanager.com https://*.google-analytics.com https://s.ytimg.com https://www.youtube.com https://www.yourdomainname.com https://*.googleapis.com https://*.gstatic.com https://*.w.org data: 'unsafe-inline' 'unsafe-eval';" always;
     add_header X-Xss-Protection "1; mode=block" always;
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header Access-Control-Allow-Origin "https://mydomen.ru";
        add_header Referrer-Policy "origin-when-cross-origin" always;
        add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
    }
}
```


```text
sudo nginx -t
sudo service nginx restart
```

> Переходим на сайт и видим экран приветствия


### Дополнительно:


> sudo nano /etc/nginx/nginx.conf

```text
        include /etc/nginx/mime.types;            (эта строка уже будет)
        default_type application/octet-stream;    (эта строка уже будет)
                     
           ниже добавить:

        client_body_buffer_size 16k;
        client_header_buffer_size 1k;
        client_max_body_size 8m;
        large_client_header_buffers 2 1k;
        server_tokens off;


```

```text
sudo nginx -t
sudo service nginx restart         (перезапуск nginx)
sudo systemctl restart gunicorn    (перезапуск гуникорн)

```

```text
Можно было и не тут указывать, но пока так. Ограничил размер буфера и скрыл подробности nginx (в ответе браузера видно)
Все проверил, все работает если что не так пишите. В файле выше там всего дофига, опишу подробно когда картина в целом прорисуется.
```
