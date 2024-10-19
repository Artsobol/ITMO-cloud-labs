# Лабораторная работа №2

Необходимо написать хороший и плохой Dockerfile

## Выполнили

Гайдук Алина (К3241), Соболев Артем (К3240)

<img src="1.jpg" alt="А также самый главный участник" width="800"/>

## Техническое задание

1. Написать “плохой” Dockerfile, в котором есть не менее трех “bad practices” по написанию докерфайлов
2. Написать “хороший” Dockerfile, в котором эти плохие практики исправлены
3. В Readme описать каждую из плохих практик в плохом докерфайле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат
4. В Readme описать 2 плохих практики по работе с контейнерами. ! Не по написанию докерфайлов, а о том, как даже используя хороший докерфайл можно накосячить именно в работе с контейнерами.

## Результат

Два Dockerfile, в одном bad practices, а в другом best practices

## Система

Windows

## Ход работы

### Реальный пример Dockerfile в проекте

Для этой Лабораторной работы мы взяли Dockerfile, который реализован в проекте одного из участников команды "Любим котов". Данный проект можно найти и посмотреть [здесь](https://github.com/Artsobol/RestfulAPI). Оттуда же мы возьмём и docker compose для лабораторной работы 2*, в котором также будет произведена сборка и запуск данного контейнера

### Dockerfile с bad practices

Возьмем этот Dockerfile и напишем плохие практики

```Dockerfile
FROM python:latest


ENV HOME=/home/fast \
    APP_HOME=/home/fast/app \
    PYTHONPATH="$PYTHONPATH:/home/fast" \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

RUN mkdir -p $APP_HOME && \
    groupadd -r fast && \
    useradd -r -g fast fast

WORKDIR $APP_HOME

RUN apt-get update && apt-get install -y \
    vim \
    net-tools \
   iputils-ping \
   libaa-bin \

ADD requirements.txt .
RUN pip install --upgrade pip && \
    pip install -r requirements.txt

ADD app app
ADD alembic.ini .

USER root

```

### Почему этот Dockerfile плохой?

**Плохие практики**:

1. `FROM python:latest` - плохая практика. Python может обновиться и докер установит новую версию, на которой работоспособность программы не проверялась. Могут возникнуть ошибки или неожиданное поведение программы

2. Установка ненужных пакетов, таких как `vim`, `iputils-ping`, `libaa-bin` и `net-tools`, которые не требуются для выполнения программы. Эти инструменты увеличивают размер образа

3. `ADD requirements.txt .`, `ADD app app`, `ADD alembic.ini .` - плохо использовать `ADD`, так как `ADD` может загружать данные по URL, распаковывать архивы

4. `USER root` - плохая практика с точки зрения безопасности. Злоумышленник может использовать root для более масштабных атак

### Dockerfile с best practices

```Dockerfile
FROM python:3.12.0-slim

ENV HOME=/home/fast \
    APP_HOME=/home/fast/app \
    PYTHONPATH="$PYTHONPATH:/home/fast" \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

RUN mkdir -p $APP_HOME \
 && groupadd -r fast\
 && useradd -r -g fast fast


WORKDIR $APP_HOME

COPY requirements.txt .

RUN pip install --upgrade pip \
 && pip install -r requirements.txt \
 && rm -rf ~/.cache/pip

COPY app app
COPY alembic.ini .

RUN chown -R fast:fast .

USER fast
```

**Хорошие практики**:

1. `FROM python:3.12.0-slim` - устанавливаем конкретную версию Python, притом уменьшенный образ, который весит меньше, что хорошо скажется на итоговом размере образа

2. Убрали установку ненужных пакетов, оставив только те, которые действительно нужны для исполнения программы. Также мы очистили кэш после установки пакетов для уменьшения размера образа

3. Вместо `ADD` мы использовали `COPY`. `COPY` копирует файлы в контейнер без дополнительных действий. Это помогает избежать автоматической распаковки или скачивание данных

4. Мы использовали `USER fast`, которому выдали нужные права доступа, что хорошо с точки зрения безопасности. Теперь у пользователя минимальные права, что не позволит злоумышленнику делать масштабные атаки

### Две плохие практики по работе с контейнерами

1. Запуск контейнеров от `root`. Это опасно, так как в случае атаки злоумышленник может получить полный доступ к системе хост-машины

2. Использовать автоматический перезапуск контейнеров. Это может создать высокую нагрузку на систему. Лучше настраивать параметры ограничения перезапуска. Допустим, `--restart on-failure`

3. Запуск контейнера без ограничения ресурсов. Контейнер может потреблять все доступные ресурсы и приводить к уменьшению производительности хост машины. Запуск контейнера с ограничение ресурсов: `docker run -d --memory 512m --cpus 0.5 myapp`

## Вывод

Мы научились работать с Docker и создавать Dockerfile. Узнали хорошие и плохие практики при написании Dockerfile, а также плохие практики при работе с Dockerfile

## Источники

[Что такое докер](https://www.youtube.com/watch?v=jVV8CVURmrE&list=PLqVeG_R3qMSwjnkMUns_Yc4zF_PtUZmB-)

[Пишем dockerfile](https://www.youtube.com/watch?v=_pmbcKuxNkk&list=PLqVeG_R3qMSwjnkMUns_Yc4zF_PtUZmB-&index=7)

<img src="2.jpg" alt="Устали" width="800"/>
