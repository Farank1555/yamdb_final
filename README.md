# Yamdb_final

[![yamdb_final workflow](https://github.com/Farank1555/yamdb_final/workflows/yamdb_final%20workflow/badge.svg)](https://github.com/Farank1555/yamdb_final/actions/workflows/yamdb_workflow.yml)

[![GitHub%20Actions](https://img.shields.io/badge/-GitHub%20Actions-464646??style=flat-square&logo=GitHub%20actions)](https://github.com/features/actions)
[![GitHub](https://img.shields.io/badge/-GitHub-464646??style=flat-square&logo=GitHub)](https://github.com/)
[![docker](https://img.shields.io/badge/-Docker-464646??style=flat-square&logo=docker)](https://www.docker.com/)
[![NGINX](https://img.shields.io/badge/-NGINX-464646??style=flat-square&logo=NGINX)](https://nginx.org/ru/)
[![Python](https://img.shields.io/badge/-Python-464646??style=flat-square&logo=Python)](https://www.python.org/)
[![Django](https://img.shields.io/badge/-Django-464646??style=flat-square&logo=Django)](https://www.djangoproject.com/)
[![PostgreSQL](https://img.shields.io/badge/-PostgreSQL-464646??style=flat-square&logo=PostgreSQL)](https://www.postgresql.org/)

Проект Yamdb_final создан для демонстрации методики DevOps (Development Operations) и идеи Continuous Integration (CI),
суть которых заключается в интеграции и автоматизации следующих процессов:
* синхронизация изменений в коде
* сборка, запуск и тестерование приложения в среде, аналогичной среде боевого сервера
* деплой на сервер после успешного прохождения всех тестов
* уведомление об успешном прохождении всех этапов

Само приложение взято из проекта [api_yamdb](https://github.com/Farank1555/api_yamdb), который представляет собой API сервиса отзывов о фильмах, книгах и музыке.
Зарегистрированные пользователи могут оставлять отзывы (Review) на произведения (Title).
Произведения делятся на категории (Category): «Книги», «Фильмы», «Музыка». 
Список категорий может быть расширен администратором. Приложение сделано с помощью Django REST Framework.

Для Continuous Integration в проекте используется облачный сервис GitHub Actions.
Для него описана последовательность команд (workflow), которая будет выполняться после события push в репозиторий.

## Начало

Клонирование проекта:
```
git clone https://github.com/Farank1555/yamdb_final.git
```
Для добавления файла .env с настройками базы данных на сервер необходимо:

* Установить соединение с сервером по протоколу ssh:
    ```
    ssh username@server_address
    ```
    Где username - имя пользователя, под которым будет выполнено подключение к серверу.
    
    server_address - IP-адрес сервера или доменное имя.
    
    Например:
    ```
    ssh practisyandex@84.252.128.243
  

* В домашней директории проекта
    Создать папку www/:
    ```
    mkdir www
    ```
    В ней создать папку yamdb_final/:
    ```
    mkdir www/yamdb_final
    ```
    В папке yamdb_final создать файл .env:
    ```
    touch www/yamdb_final/.env
    ```

* Добавить настройки в файл .env:
    ```
    sudo nano www/yamdb_final/.env
    ```
    Пример добавляемых настроек:
    ```
    DB_ENGINE=django.db.backends.postgresql
    DB_NAME=postgres
    POSTGRES_USER=postgres
    POSTGRES_PASSWORD=postgres
    DB_HOST=postgres
    DB_PORT=5432
    ```

Также необходимо добавить Action secrets в репозитории на GitHub в разделе settings -> Secrets:
* DOCKER_PASSWORD - пароль от DockerHub;
* DOCKER_USERNAME - имя пользователя на DockerHub;
* HOST - ip-адрес сервера;
* SSH_KEY - приватный ssh ключ (публичный должен быть на сервере);
* TELEGRAM_TO - id своего телеграм-аккаунта (можно узнать у @userinfobot, команда /start)
* TELEGRAM_TOKEN - токен бота (получить токен можно у @BotFather, /token, имя бота)

### Проверка работоспособности

Теперь если внести любые изменения в проект и выполнить:
```
git add .
git commit -m "..."
git push
```
Комманда git push является триггером workflow проекта.
При выполнении команды git push запустится набор блоков комманд jobs (см. файл yamdb_workflow.yaml).
Последовательно будут выполнены следующие блоки:
* tests - тестирование проекта на соответствие PEP8 и тестам pytest.
* build_and_push_to_docker_hub - при успешном прохождении тестов собирается образ (image) для docker контейнера 
и отправлятеся в DockerHub
* deploy - после отправки образа на DockerHub начинается деплой проекта на сервере.
Происходит копирование следующих файлов с репозитория на сервер:
  - docker-compose.yaml, необходимый для сборки трех контейнеров:
    + postgres - контейнер базы данных
    + web - контейнер Django приложения + wsgi-сервер gunicorn
    + nginx - веб-сервер
  - nginx/default.conf - файл кофигурации nginx сервера
  - static - папка со статическими файлами проекта
  
  После копировния происходит установка docker и docker-compose на сервере
  и начинается сборка и запуск контейнеров.
* send_message - после сборки и запуска контейнеров происходит отправка сообщения в 
  телеграм об успешном окончании workflow

После выполнения вышеуказанных процедур необходимо установить соединение с сервером:
```
ssh username@server_address
```
Отобразить список работающих контейнеров:
```
sudo docker container ls
```
В списке контейнеров копировать CONTAINER ID контейнера username/yamdb_final_web:latest (username - имя пользователя на DockerHub):
```
CONTAINER ID   IMAGE                            COMMAND                  CREATED          STATUS          PORTS                NAMES
111111111111   nginx:1.19.6                     "/docker-entrypoint.…"   50 minutes ago   Up 50 minutes   0.0.0.0:80->80/tcp   yamdb_final_nginx_1
222222222222   username/yamdb_final_web:latest  "/bin/sh -c 'gunicor…"   50 minutes ago   Up 50 minutes                        yamdb_final_web_1
333333333333   postgres:13.1                    "docker-entrypoint.s…"   50 minutes ago   Up 50 minutes   5432/tcp             yamdb_final_postgres_1
```
Выполнить вход в контейнер:
```
sudo docker exec -it 222222222222 bash
```
Внутри контейнера выполнить миграции:
```
python manage.py migrate
```
Также можно наполнить базу данных начальными тестовыми данными:
```
python3 manage.py shell
>>> from django.contrib.contenttypes.models import ContentType
>>> ContentType.objects.all().delete()
>>> quit()
python manage.py loaddata fixtures.json
```
Теперь проекту доступна статика. В админке Django (http://<server_address>/admin)
доступно управление данными. Если загрузить фикструры, то будет доступен superuser:
* email: admin5@admin
* password: admin

Для создания нового суперпользователя можно выполнить команду:
```
$ python manage.py createsuperuser
```
и далее указать: 
```
Email:
Username:
Password:
Password (again):
```
Для обращения к API проекта:


* http://84.252.128.243/api/v1/auth/token/
* http://84.252.128.243/api/v1/users/
* http://84.252.128.243/api/v1/categories/
* http://84.252.128.243/api/v1/genres/
* http://84.252.128.243/api/v1/titles/
* http://84.252.128.243/api/v1/titles/{title_id}/reviews/
* http://84.252.128.243/api/v1/titles/{title_id}/reviews/{review_id}/
* http://84.252.128.243/api/v1/titles/{title_id}/reviews/{review_id}/comments/

Cписок и подробное описание доступных запросов к приложению можно посмотреть:
* http://84.252.128.243/redoc/

Для остановки и удаления контейнеров и образов на сервере:
```
sudo docker stop $(sudo docker ps -a -q) && sudo docker rm $(sudo docker ps -a -q) && sudo docker rmi $(sudo docker images -q)
```
Для удаления volume базы данных:
```
sudo docker volume rm yamdb_final_postgres_data
```

P.S. Если что-то пошло не так то выполните ребилд:
```
docker-compose up --build
```
Это пересоберет докер






## Автор

* **Бесчастнов Иван**







PI_YamDB
REST API для сервиса YaMDb — базы отзывов о фильмах, книгах и музыке.

Описание
Проект YaMDb собирает отзывы пользователей на произведения. Произведения делятся на категории: «Книги», «Фильмы», «Музыка». Произведению может быть присвоен жанр. Новые жанры может создавать только администратор. Читатели оставляют к произведениям текстовые отзывы и выставляют произведению рейтинг (оценку в диапазоне от одного до десяти). Из множества оценок автоматически высчитывается средняя оценка произведения.

Стек технологий
проект написан на Python с использованием Django REST Framework
библиотека Simple JWT - работа с JWT-токеном
библиотека django-filter - фильтрация запросов
базы данны - SQLite3 и PostgreSQL
автоматическое развертывание проекта - Docker, docker-compose
система управления версиями - git
Как запустить проект, используя Docker (база данных PostgreSQL):
Клонируйте репозитроий с проектом:
git clone https://github.com/netshy/api_yamdb/
В директории проекта создайте файл .env, по пути project_name/api_yatube/.env, в котором пропишите следующие переменные окружения (для тестирования можете использовать указанные значения переменных):
DB_ENGINE=django.db.backends.postgresql
DB_NAME=postgres
DB_USER=postgres
DB_PASSWORD=postgres
DB_HOST=db
DB_PORT=5432

С помощью Dockerfile и docker-compose.yaml разверните проект:
docker-compose up --build

Автоматически были созданы миграции для приложения и произошла миграция в БД. Загружены тестовые данные.
Ваш проект запустился на http://127.0.0.1/
Полная документация доступна по адресу http://127.0.0.1/redoc/
С помощью команды

docker exec -ti <container_id> pytest
можете запустить тесты и проверить работу модулей
А командами
docker-compose exec web python manage.py migrate --noinput
docker-compose exec web python manage.py createsuperuser
docker-compose exec web python manage.py collectstatic --no-inpu

можно создавать миграции, суперпользователся и статику


Алгоритм регистрации пользователей
Пользователь отправляет запрос с параметрами email и username на /auth/email/.
YaMDB отправляет письмо с кодом подтверждения (confirmation_code) на адрес email .
Пользователь отправляет запрос с параметрами email и confirmation_code на /auth/token/, в ответе на запрос ему приходит token (JWT-токен).
Ресурсы API YaMDb
Ресурс AUTH: аутентификация.
Ресурс USERS: пользователи.
Ресурс TITLES: произведения, к которым пишут отзывы (определённый фильм, книга или песня).
Ресурс CATEGORIES: категории (типы) произведений («Фильмы», «Книги», «Музыка»).
Ресурс GENRES: жанры произведений. Одно произведение может быть привязано к нескольким жанрам.
Ресурс REVIEWS: отзывы на произведения. Отзыв привязан к определённому произведению.
Ресурс COMMENTS: комментарии к отзывам. Комментарий привязан к определённому отзыву.
Пример http-запроса (POST) для создания нового комментария к отзыву:

Если хотите в ручную то:
### Установка на локальной машине:
* Клонировать репозиторий
  >*python3 -m venv venv*
* Активировать виртуальное окружение
  >*source venv/bin/activate*
* Установить зависимости
  >*pip install -r requirements.txt*
 