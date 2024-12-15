# Анатомия `Dockerfile`

## Плохой `Dockerfile`
Начнем с примера плохого докерфайла, который часто можно встретить в проектах:
```Dockerfile
FROM ubuntu:latest
ADD . /app
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN pip3 install flask
RUN pip3 install redis
ENV SECRET_KEY=mysupersecretkey123
EXPOSE 5000
CMD python3 /app/app.py
```

Но почему же он плохой?

## Проблемы
### Первая
В самом начале можем заметить тег `latest`, а это же вообще лотерея, сегодня работает а завтра отпуск
Так еще и убунту тяжелая, ну зачем!

*Как сделать то надо?* А вот так:
```docker
FROM python:3.9-slim-buster
```
Ведь проще взять конкретную версию Python с легковесным дистрибутивом Debian, и вообще топово

### Вторая
Если внимательно присмотреться, можно заметить ПЯТЬЬ команд RUN, а ведь каждая создает новый слой в образе, что тупо -> тяжело -> медленно

А вот как надо:
```docker
RUN apt-get update && \
    apt-get install -y python3 python3-pip && \
    rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
```

### Третья
Ну это вообще позор, так нельзя делать
У нас же захардкожен секрет, это нарушает все законы и правила

Надо исправить:
```
ENV SECRET_KEY=${SECRET_KEY}
```


## ХОРОШИЙ `Dockerfile`
Вот как должен выглядеть правильный Dockerfile:
```docker
FROM python:3.9-slim-buster

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV SECRET_KEY=${SECRET_KEY}

USER nobody
EXPOSE 5000

CMD ["python3", "app.py"]
```

### Давайте разберем апгрейды:
- Используем официальный Python образ определенной версии
- Копируем сначала requirements.txt, чтобы лучше использовать кэш докера
- Запускаем приложение от непривилегированного пользователя
- Ну и используем переменные из окружения для секретов

## Проблемы при работе с хорошими докерфайлами

### 1 Ресурсы
Зачем давать контейнеру бесконечные ресурсы, а вдруг мемори лик и все
Ну вот что мы можем сделать:
Максимум памяти:
```zsh
--memory 5m
```
Максимум файла подкачки:
```zsh
--memory-swap 100m
```
CPU limit:
```zsh
--cpus 2.5
```

### 2 Уполномоченный
1. `privileged` запуск контейнера
По доке, такой контейнер получит god mode и запустит докер в докере


А как надо?
А просто разрешим только то, что нужно, а что не нужно - запретим
*thoughtful*

Чтобы этого достичь прибегнем с использования `--cap-drop` и `--cap-add`
```zsh
docker run
--cap-drop NET_ADMIN
--cap-drop SYS_MODULE
--cap-add SYS_TIME
-ti lab1 /bin/sh
```

# ✨🌟🤩

## `Docker Compose` и распространенные ошибки
Начнем с плохого примера:
```yml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    environment:
      - SECRET_KEY=mysupersecretkey567
    volumes:
      - .:/app
    privileged: true
    
  redis:
    image: redis:latest
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    restart: always

volumes:
  redis_data:
```

## Проблемы
### Первая:
`privileged: true` дает контейнеру полный доступ к хост-системе, а так не надо делать

А чтобы исправить мы можем просто использовать read_only: true и минимальные необходимые capabilities

### Вторая:
Можно заметить, что порты `Redis` доступны извне

А чтобы исправить надо использовать `expose` вместо `ports` для внутренней коммуникации

### Третья:
Мы монтируем весь проект из-за `.:/app`

Ну и очевидно надо монтировать отдельные файлы!

## Исправленный "Хороший" `docker-compose`
```yml
version: '3.8'
services:
  web:
    build: 
      context: .
      dockerfile: Dockerfile
    ports:
      - "127.0.0.1:5000:5000"
    environment:
      - SECRET_KEY=${SECRET_KEY}
    read_only: true
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    
  redis:
    image: redis:7.0-alpine
    expose:
      - "6379"
    volumes:
      - redis_data:/data:ro
    user: redis
    networks:
      - backend

volumes:
  redis_data:

networks:
  backend:
    internal: true
```

# Изоляция контейнеров
Чтобы все хорошо работало, стоит позаботиться об изоляции сервисов!

## Примеры:
Внутренние сети (помечаем сеть как `internal`)
```yml
networks:
  backend:
    internal: true
```

Доступ к портам (вместо `ports` укажем какой нибудь хард порт)
```yml
expose:
  - "5677"
```

Наименьшие привилегии (все и так понятно)
```yml
security_opt:
  - no-new-privileges:true
cap_drop:
  - ALL
```

# Заключение
Вспомнил что такое докер и почему он мне не нравится! Но в целом лаба оч прикольная (говорю не чтоб подмазаться, а реально формат good/bad practices заставляет прям покопаться и вникнуть). мне понрав