![example workflow](https://github.com/k0sdm1/kittygram_finaL/actions/workflows/main.yml/badge.svg)

# Kittygram — место где можно поделиться фотографиями кошек и их достижениями.
## Описание проекта
После регистрации появляется возможность выкладывать фоторагфии с описанием и списком достижений, а также просматривать публикации бругих пользователей.

## Развертывание
Для установки необходимо выполнить следующие действия:

0. Установить GitBash, DockerDesktop если это не Linux. Все равно установить Docker, если это Linux. (https://www.docker.com/get-started/)

1. Клонировать репозиторий:
   ```
   git clone https://github.com/k0sdm1/kittygram_final.git
   ```
2. Создать файл .env со следующим набором данных:
   ```
    POSTGRES_USER=     пользователь БД
    POSTGRES_PASSWORD= пароль доступа к БД
    POSTGRES_DB=       название БД
    DB_HOST=           докер-контейнер БД
    DB_PORT= 5432      порт можно оставить стандартным
    SECRET_KEY=        секретный ключ django бэкэнда
    DEBUG=             Переключатель режима отладки django бэкэнда (true для режима отладки)
    ALLOWED_HOSTS=     127.0.0.1,localhost... Хосты с доспупом к бэкэнду (добавить ip сервера и url)
    IS_SQLITE=0        Для отладки значение 1 переключит БД по умолчанию на sqlite, 0 для Postgres
   ```
3. Создать образы докера и загрузить из на docker hub, {usrnm} = Ваше имя пользователя на DockerHub:
   ```
   cd frontend
   docker build -t {usrnm}/kittygram_frontend .
   cd ../backend
   docker build -t {usrnm}/kittygram_backend .
   cd ../nginx
   docker build -t {usrnm}/kittygram_gateway .

   docker push {usrnm}/kittygram_frontend
   docker push {usrnm}/kittygram_backend
   docker push {usrnm}/kittygram_gateway
   ```
4. Подключиться к удаленному серверу:
   ```
   ssh -i путьДоКлючаSSH/имяПриватногоSSHКлюча имяПользователяНаСервере@адресСервера
   ```   
5. Создать папку проекта и установить Docker Compose:
   ```
   mkdir kittygram

   sudo apt update
   sudo apt install curl
   curl -fsSL https://get.docker.com -o get-docker.sh
   sudo sh get-docker.sh
   sudo apt install docker-compose
   ```
6. Скопировать файл docker-compose.production.yml и .env:
   ```
   cd путьДоЛокальногоРепозитория
   scp -i путьДоКлючаSSH/имяПриватногоSSHКлюча docker-compose.production.yml имяПользователяНаСервере@адресСервера:/home/имяПользователяНаСервере/kittygram/docker-compose.production.yml
   scp -i путьДоКлючаSSH/имяПриватногоSSHКлюча .env имяПользователяНаСервере@адресСервера:/home/имяПользователяНаСервере/kittygram/.env
   ```
7. Запустить демон Docker Compose на сервере:
   ```
   cd путьДоKittygram
   sudo docker compose -f docker-compose.production.yml up -d
   ```
8. Выполнить миграции и собрать статику:
   ```
   sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
   sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic
   sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
   ```
9. Настроить маршрутизацию nginx (установить nginx если его нет: ```sudo apt install nginx -y ; sudo systemctl start nginx``` ):
   ```
   sudo nano /etc/nginx/sites-enabled/default

   # В открывшемся редактрое вставить:
   server {
    server_name ip url;
      location / {
       proxy_set_header Host $http_host;
       proxy_pass http://127.0.0.1:9000;
      }
   }
   
   # Проверить конфиг nginx:
   sudo nginx -t
   
   # Ответ вида:
   nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
   nginx: configuration file /etc/nginx/nginx.conf test is successful
   # означает что ошибок нет и можно перезапустить сервис:
   sudo systemctl reload nginx
   ```
10. Для CI/CD:
    Файл для Gihub Actons находится в репозитории:
    ```
    kittygram/.github/workflows/main.yml
    ```
    Неоходимо настроить секреты в Github Actions:
    ```
    DOCKER_USERNAME                # имя пользователя в DockerHub
    DOCKER_PASSWORD                # пароль пользователя в DockerHub
    HOST                           # IP-адрес сервера
    USER                           # имя пользователя
    SSH_KEY                        # содержимое приватного SSH-ключа (cat ~/.ssh/названиеКлюча)
    SSH_PASSPHRASE                 # пароль для SSH-ключа
      
    TELEGRAM_TO                    # ID телеграм-аккаунта куда сообщать о результате (узнать у @userinfobot, команда /start)
    TELEGRAM_TOKEN                 # токен бота (получить токен @BotFather, команда /token, имя бота)
    ```
## Использованные технологии и сервисы

Python 3.9.10, Django 3.2.3, djangorestframework==3.12.4, PostgreSQL 13.10, Docker, Github Actions

## Автор

Автоматизация развертывания: [[ссылка](https://github.com/k0sdm1)]("k0sdm1")
