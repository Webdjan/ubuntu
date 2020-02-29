### Nginx и Ubuntu 18.04 настройка:

* Заходим по ssh с настроенными правами sudo:

``` 
Настройка пользователя с правами sudo тут: 
https://github.com/Webdjan/ubuntu/blob/master/ssh-user.md
```

1. Обновляем локальный индекс пакетов
2. Установка Nginx со всеми зависимостями
3. Список конфигураций брандмауэра
4. Разрешаем трафик через порт 80
5. Проверяем изменения

```text
1)	sudo apt update
2)	sudo apt install nginx
3)	sudo ufw app list
4)      sudo ufw allow 'Nginx HTTP'
5)      sudo ufw status
```

### Управление

```text
curl -4 yourdomen.ru            (узнать ip)
systemctl status nginx          (проверка вебсервера)
sudo ufw status                 (проверка статуса брандмауэра)
sudo systemctl stop nginx       (остановка)
sudo systemctl start nginx      (запуск) 
sudo systemctl restart nginx    (остановка и запуск)
sudo systemctl reload nginx     (перезагрузка без разрыва соединения)
sudo systemctl disable nginx    (отключить автозагрузку)
sudo systemctl enable nginx     (включить автозагрузку
```

### Настройка блоков

```text
sudo mkdir -p /var/www/mydomen.ru/html               (создаем каталог)
sudo chown -R $USER:$USER /var/www/mydomen.ru/html   (назначаем владельца каталога)
sudo chmod -R 755 /var/www/mydomen.ru                (права доступа к корню)
nano /var/www/mydomen.ru/html/index.html             (создаем простую страничку html с базовой структурой)
```

1.  Не изменяя дефолтный конфиг, создадим новый
2.  Включение файла
3.  Раскомментируем строку erver_names_hash_bucket_size 64 убрав #
4.  Проверка наличия ошибок
5.  Перезапуск nginx

```text
1)  sudo nano /etc/nginx/sites-available/mydomen.ru

server {
        listen 80;
        listen [::]:80;

        root /var/www/mydomen.ru/html;
        index index.html index.htm index.nginx-debian.html;

        server_name mydomen.ru www.mydomen.ru;

        location / {
                try_files $uri $uri/ =404;
        }
}
```

```text
2)  sudo ln -s /etc/nginx/sites-available/mydomen.ru /etc/nginx/sites-enabled/
3)  sudo nano /etc/nginx/nginx.conf   (инфо выше под пунктом 3)
4)  sudo nginx -t
5)  sudo systemctl restart nginx
```


> Nginx теперь должен обслуживать ваше доменное имя. Переходим на свой домен и видим то, что написали в
> index.html

### Настройка закончена! Дополнительное инфо:

```text
/var/www/html                   (дефолтная страница nginx)
/etc/nginx                      (каталог конфигов)
/etc/nginx/nginx.conf           (основной файл конфига)
/etc/nginx/sites-available/     (хранилище блоков)
/etc/nginx/sites-enabled/       (хранилище включенных блоков)
/etc/nginx/snippets             (фрагменты конфигурации, хз еще не смотрел)
/var/log/nginx/access.log       (лог запросов)
/var/log/nginx/error.log        (лог ошибок)
```

```text
/etc/nginx/sites-available/mydomen.ru    (наш конфиг серверного блока NGINX)
/var/www/mydomen.ru/html/                (своя структура каталогов)
/var/www/mydomen.ru/html/index.html      (своя базовая html страничка)
sudo systemctl restart nginx             (команда перезапуска NGINX)
```
