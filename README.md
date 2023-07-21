#  Kittygram Social Network
## Запуск в контейнере DockerHub и организация CI/CD с помощью GitHub Actions

### 1. Создание и развертка контейнеров проекта:

Форкнуть репозиторий, сконировать и перейти в рабочкорневую папку `kittygram_final`

Переименовать .env.example в .env и заполнить по шаблону:

```
mv .env.example .env
nano .env
```

Убедиться, что сервис (демон) Docker активен в системе:

```
# Для Linux
sudo systemctl status docker
# Для Windows и MacOS вызвать диспетчер задач и убедиться, что процесс(ы) Docker Desctop запущен(ы)
```

Развернуть контейнерное пространство:

(!) если развертка происходит впервые:

- необходимо открыть docker-compose.yml и запенить для каждого из контейнеров `backend`,  `frontend`, `gateway` способ выбора образа с `image` на `build` - таким образом будут созданы исходные docker образы проекта
- впоследствии возможно создать свои образы `backend`,  `frontend`, `gateway` и использовать параметр `image`

```
nano docker-compose.yml
```
```
....
backend:
   ...
   # эту строку строку ниже:
   image: <username>/kittygram_backend
   # заменить на:
   build: ./backend/
   ...
frontend:
   ...
   # эту строку строку ниже:
   image: <username>/kittygram_frontend
   # заменить на:
   build: ./frontend/
   ...
gateway:
   ...
   # эту строку строку ниже:
   image: <username>/kittygram_gateway
   # заменить на:
   build: ./nginx/
   ...
```

Развернуть контейнерную группу в виде фоновоного демона:

```
docker compose up -d
```

Настроить миграции backend:

```
docker compose exec backend python manage.py makemigrations cats
docker compose exec backend python manage.py migrate
```

Передать статику в Nginx:

```
docker compose exec backend python manage.py collectstatic
docker compose exec backend cp -r /kittygram/collect_static/. /backend_static/static/
```

Настроить Ваш сервер на отпраку запросов к сайту Kittygram на порт 9000 (согласно настройке образа `gateway`).
   
2. Создания CI/CD на GitHub Actions

Создать в корне проекта скрытую директиву для размещения файла с интсрукциями для GitHub Actions:

```
mkdir .github/workflows
```

Переместить файл `kittygram_workflow.yml` в созданную директорию:

```
mv kittygram_workflow.yml ./.github/workflows/kittygram_workflow.yml
```

Перейти в раздел с проектом на GutHub -> Settings -> Secrets and variables -> actions

В разделе `secrets` создать (кнопка `New repository secret`) следующие "секреты":
- DOCKERHUB_USERNAME - имя пользователя DockerHub аккаунта
- DOCKERHUB_PASSWORD - пароль от DockerHub аккаунта
- SSH_HOST - ваш публичный IP сервера*
- SSH_KEY - SSH-ключ доступа к серверу*, начиная с `-----BEGIN OPENSSH PRIVATE KEY-----` и заканчивая `-----END OPENSSH PRIVATE KEY-----`
- SSH_USERNAME - имя пользователя учетной записи на сервере*
- SSH_PASSPHRASE - пароль учетной записи на сервере*
- TELEGRAM_BOT_TOKEN - токен Telegram Bot**, который будет уведомлять об успешном деплое проекта на сервере
- TELEGRAM_ME_ID - ID пользователя Telegram, которому бот будет присылать сообщения

\* под сервером имеется ввиду сервер, на котором разворачивается Kittygram
\*\* если уведомления в телеграм не нужны, необходимо из `kittygram_workflow.yml` удалить задание `send_message_telegram`

3. Результаты работы

Теперь при каждом обновлении ветки main репозитория:
- запустятся автотесты приложения `backend`
- автоматически обновятся контейнеры `backend`, `frontend`, `gateway`
- контейнеры отправятся на DockerHub облако
- контейнеры скачаются с DockerHub облако на сервер и перезапустятся

___

### LICENCE

MIT

**Free Software, Hell Yeah!**

Deployment instructions by [TheSuncatcher222](https://github.com/TheSuncatcher222)
