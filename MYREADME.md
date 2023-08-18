Содержимое
1. Dockerfile
2. entrypoint.sh
3. .env
4. settings.py
5. Docker-compose
6. Создание и запуск контейнера

Нужен gunicorn, psycopg2-binary, django-environ (переменные окружения).
# Dockerfile
```dockerfile
FROM python:3.10-alpine

ENV PYTHONDONTWRITEBYTECODE=1 PYTHONUNBUFFERED=1

COPY . .

WORKDIR /app

RUN apk --update --no-cache --virtual .tmp-build-deps gcc libc-dev linux-headers postgresql-dev && pip install --no-cahce-dir -r requirements.txt
```
`alpine`  – это легкий дистрибутив Linux, разработанный для обеспечения безопасности и эффективности использования ресурсов.

`PYTHONDONTWRITEBYTECODE=1` - запрещает записывать файлы `.pyc` на диск.

`PYTHONUNBUFFERED=1` - запрещает буферизацию. Не будет буферизировать вывод консоли.

`RUN` - установка пакетов для работы с postgresql и установка пакеты python.
# entrypoint.sh
```sh
#!/usr/bin/env sh


python manage.py makemigrations

python manage.py migrate

gunicorn shop.wsgi:application --bind 0.0.0.0:8000 --reload -w 4

```
# .env
```env
DEBUG=1  
SECRET_KEY=
  
# DATABASE CREDENTIALS  
POSTGRES_HOST=postgresql  
POSTGRES_DB=postgres  
POSTGRES_USER=postgres  
POSTGRES_PASSWORD=password  
POSTGRES_PORT=5432
```
`POSTGRES_DB=postgres`, `POSTGRES_USER=postgres`, `POSTGRES_PASSWORD=password` идут по умолчанию нам не надо будет ничего создавать. 
# settings.py
```python
import environ

# инсталяция
env = environ.Env()  
environ.Env.read_env('.env')

SECRET_KEY = env("SECRET_KEY")  
  
DEBUG = env("DEBUG")

DATABASES = {  
	'default': {  
	'ENGINE': 'django.db.backends.postgresql_psycopg2',  
	'NAME': env("POSTGRES_DB"),  
	'USER': env("POSTGRES_USER"),  
	'PASSWORD': env("POSTGRES_PASSWORD"),  
	'HOST': env("POSTGRES_HOST"),  
	'PORT': env("POSTGRES_PORT")  
	}  
}
```
# Docker-compose
```yaml
version: "3.7"

services:
	web:
	    build:
			context: .
			dockerfile: Dockerfile
		ports:
			- "8000:8000"
		entrypoint:  
			- ./entrypoint.sh
	  
	postgresql:
		image: postgres
		environment:
			- POSTGRES_PASSWORD=password
		ports:
			- "5432:5432"
```

# Создание и запуск контейнера
## Создание контейнера
```cmd
docker-compose build
```
## Запуск контейнера
```cmd
docker-compose up -d
```
