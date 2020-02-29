## Шаблон для теста, делаем на ПК (Linux по умолчанию python3.6.9 Джанго 3)

> 1) Подготовка:

```text
sudo apt update
sudo -H pip3 install --upgrade pip
sudo -H pip3 install virtualenv
mkdir ~/world
mkdir ~/world/rif
cd ~/world/rif
pwd 								(home/neyton/world/rif)
python3 -m venv myprojectenv		(виртуальное окружение)
mkdir myprojectdir					(папка для проекта)
```

> 2) Переходим в виртуальное окружение:

```text
source myprojectenv/bin/activate
pip install django
pip install Pillow
django-admin.py startproject myproject ~/world/rif/myprojectdir
```

> 2.1) пояснение

```text
~/world/rif/                                    	(Виртуальное окружение тут)  
~/world/rif/myprojectdir/                       	(тут manage.py)  
~/world/rif/myprojectdir/myproject/             	(тут settings.py)
		
					Полный путь:

home/neyton/world/rif/                              (Виртуальное окружение тут)  
home/neyton/world/rif/myprojectdir/                 (тут manage.py)  
home/neyton/world/rif/myprojectdir/myproject/       (тут settings.py)
```

> 3) 

```text
cd ~/world/rif/myprojectdir/
python manage.py startapp myblog				(приложение для блога)
mkdir templates									        (папка для файлов html)
mkdir static								          	(папка для стилей, скриптов и т.д.)
```

> 3.1) Пояснение:

```text
Берем шаблон html. В моем случае шаблон состоит из: 
index.html blog.html post.html и папки: css, js, img и т.д. (на основе mdbootstrap)

а) Все файлы html копируем в папку templates
b) Все папки css, js и т.д копируем в папку static
```

> 4) myproject/settings.py

```python

. . .

INSTALLED_APPS = [

    'myblog',
]

. . .

TEMPLATES = [
    
        'DIRS': [os.path.join(BASE_DIR, 'templates')],


. . .

LANGUAGE_CODE = 'ru'
TIME_ZONE = 'Europe/Moscow'

. . .

STATIC_URL = '/static/'
STATIC_DIR = os.path.join(BASE_DIR, 'static/')
STATICFILES_DIRS = [STATIC_DIR]
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media/')

```

> 5) myproject/urls.py

```python
from django.conf import settings
from django.conf.urls.static import static

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('myblog.urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

> 6) Создаем файл myblog/urls.py

```python
from django.urls import path
from myblog.views import index, blog, post


urlpatterns = [
	path('', index),
	path('blog/', blog),
	path('post/', post),
]
```

> 7) myblog/views.py

```python
from django.shortcuts import render



def index(request):
    return render(request, 'index.html', {})

def blog(request):
    return render(request, 'blog.html', {})

def post(request):
    return render(request, 'post.html', {})
```

> 8) Подготовка и запуск

```text
python manage.py makemigrations
python manage.py migrate
python manage.py runserver 
```

> 8.1) Пояснение 

```text
После запуска сервера, сайт будет доступен по  
http://127.0.0.1:8000/ , можно будет увидеть каждый шаблон, 
но еще без подключенных скриптов и стилей: 
http://127.0.0.1:8000/blog/ http://127.0.0.1:8000/post/
Если все ок, идем дальше. ctrl+c (остановить сервер)
```

### Работа с шаблонами в папке templates

> 9) Открываем файл index.html blog.html post.html и добавляем в самый верх:

```text
{% load static %}
<html>
	. . . . . . .
```

> 9.1) В этих же файлах редактируем все пути для всех картинок, стилей, скрипов и т д.  
> Во всех шаблонах: index.html blog.html post.html (для практики будет полезно)  
> Было:

```html
<link rel="stylesheet" href="css/mystyle.css">
<img src="img/logo.png" alt="">
<script type="text/javascript" src="js/bootstrap.min.js"></script>
```
> Стало {% static 'путь' %}

```html
<link rel="stylesheet" href="{% static 'css/mystyle.css' %}">
<img src="{% static 'img/logo.png' %}" alt="">
<script type="text/javascript" src="{% static 'js/bootstrap.min.js' %}"></script>
```

> 10) Проверяем все страницы после выполнения команд указанных ниже 
> http://127.0.0.1:8000/ http://127.0.0.1:8000/blog/ http://127.0.0.1:8000/post/

```text
pip freeze > requirements.txt 		(создаем файл с зависимостями)
python manage.py collectstatic
python manage.py runserver
```

> Шаблон готов для теста на реальном сервере
