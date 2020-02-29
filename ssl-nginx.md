### SSL настройка (Nginx, Ubuntu 18.04):

* Заходим по ssh с настроенными правами sudo и сервером nginx:
``` 
a) Настройка пользователя с правами sudo тут: 
https://github.com/Webdjan/ubuntu/blob/master/ssh-user.md
b) Настройка NGINX тут:
https://github.com/Webdjan/ubuntu/blob/master/nginx.md
```

```text
В целом вся информация по автоустановке сертификата тут:
https://certbot.eff.org/ Нужно лишь выбрать из фильтра на сайте
Nginx и Ubuntu 18.04
```

### Подготовка

```text
sudo apt-get update
sudo apt-get install software-properties-common
sudo add-apt-repository universe
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
```

### Установка и настрокак

1.  Установка Certbot
2.  Проверить что в файле указаны домены (server_name).
3.  Проверяем на ошибки
4.  Если вносим изменения в пункте 2

```text
1)  sudo apt-get install certbot python-certbot-nginx
2)  sudo nano /etc/nginx/sites-available/mydomen.ru
3)  sudo nginx -t
4)  sudo systemctl reload nginx
```

### Настройка брандмауэра

1.  Проверяем статус (для nginx открыт только - Nginx HTTP)
2.  Разрешаем всё для Nginx
3.  Удаляем Nginx HTTP
4.  Проверяем статус (Nginx Full) 

```text
1) sudo ufw status
2) sudo ufw allow 'Nginx Full'
3) sudo ufw delete allow 'Nginx HTTP'
```

### Получаем сертификат

```text
1)  sudo certbot --nginx -d mydomen.ru -d www.mydomen.ru

    После запуска нужно ввести адрес эл. почты и согласится с
    правилами... A(gree):  A и Enter и выбрать пункт 2.
    Если все ок, то все настройки будут сделаны автоматически.

2)  sudo certbot renew --dry-run

    Включаем автообновление сертификата
```
