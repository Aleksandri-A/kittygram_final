# О проекте
 Kittygram — социальная сеть для обмена фотографиями любимых питомцев. Это полностью рабочий проект, который состоит из бэкенд-приложения на Django и фронтенд-приложения на React.

## Настройка приложения

Клонировать репозиторий и перейти в него в командной строке:

```
git clone git@github.com:Aleksandri-A/kittygram_final.git
```

Скачайте и установите curl — консольную утилиту, которая умеет скачивать файлы по команде пользователя:
```
sudo apt update
sudo apt install curl
```

Запустите скрипт:
```
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
```

Дополнительно к Docker установите утилиту Docker Compose:
```
sudo apt-get install docker-compose-plugin 
```

В терминале в папке с docker-compose.yml выполните команду:

```
docker compose up 
```

Перейдите в директорию, где лежит файл docker-compose.yml, и выполните миграции:
```
docker compose exec backend python manage.py migrate
```

Выполните команду сборки статики. После этого выполните команду копирования собранных файлов в /backend_static/static/ — перенесите файлы не в /backend_static/, а во вложенную папку: так адреса файлов для Nginx совпадут с адресами статических файлов, которые ожидает Django-проект.
```
# Собрать статику Django
docker compose exec backend python manage.py collectstatic
# Статика приложения в контейнере backend 
# будет собрана в директорию /app/collected_static/.

# Теперь из этой директории копируем статику в /backend_static/static/;
# эта статика попадёт на volume static в папку /static/:
docker compose exec backend cp -r /app/collected_static/. /backend_static/static/ 
```

## Деплой: публикация проекта в Docker на сервере

Поочерёдно выполните на сервере команды для установки Docker и Docker Compose для Linux.

```
sudo apt update
sudo apt install curl
curl -fSL https://get.docker.com -o get-docker.sh
sudo sh ./get-docker.sh
sudo apt-get install docker-compose-plugin 
```

Создайте на сервере директорию kittygram и файл docker-compose.production.yml и скопируйте в него сожержимое из локального docker-compose.production.yml.

Создайте файл .env и внесите ваши данные:
```
POSTGRES_DB
POSTGRES_USER
POSTGRES_PASSWORD
DB_PORT
DB_HOST
SECRET_KEY
ALLOWED_HOSTS
```
Для запуска Docker Compose в режиме демона команду выполните эту команду на сервере в папке kittygram/:
```
sudo docker compose -f docker-compose.production.yml up -d 
```

Выполните миграции, соберите статические файлы бэкенда и скопируйте их в /backend_static/static/:
```
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/ 
```

Если Nginx ещё не установлен на удалённый сервер, установите его:
```
sudo apt install nginx -y
```

Запустите Nginx командой:
```
sudo systemctl start nginx
```

На сервере в редакторе nano откройте конфиг Nginx и обновите настройки: 
```
nano /etc/nginx/sites-enabled/default
```

```
server {
    listen 80;
    server_name ваш_домен;
    location / {
        proxy_set_header Host $http_host;
        proxy_pass http://127.0.0.1:9000;
    }
}
```

Чтобы убедиться, что в конфиге нет ошибок — выполните команду проверки конфигурации:
```
sudo nginx -t 
```

Перезагрузите конфиг Nginx:
```
sudo service nginx reload 
```


