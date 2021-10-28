## Система развёртывания проектов Django + PostgreSQL + Traefik

Требования:

1. OC Ubuntu 20.04

## Процесс общей настройки

1. Склонировать репозиторий и войти в него `cd robost`
2. Создать виртуальное окружение `python -m venv .venv`
3. Установить зависимости `. .venv/bin/pip install -r playbook/requirements.txt`
4. Перейти в каталог `playbook`
5. Активировать `. ../.venv/bin/activate`
6. В файле `hosts.yml` подставить данные согласно комментариям: Имя пользователя, hostname/IPaddress
7. В файле `roles/docker/files/manage/db_config.yml` изменить пароль для доступа к БД (имя пользователя и начальную БД тажке можно сменить)
8. Запустить выполнение командой `ansible-playbook -i hosts.yml site.yml -kK` нужно будет ввести пароль пользователя
   Итог: будут установлены необходимые пакеты, виртуальное окружение и необходимые плейбуки для управления

## Процесс работы с системой

### Общая настройка

1. Зайти на сервер, перейти в пользователя `root` команда `sudo su -`
2. Запуск БД осуществляется командой `/root/.venv/bin/ansible-playbook /root/manage/db.yml --tags start`
3. Запуск LoadBalancer командой `/root/.venv/bin/ansible-playbook /root/manage/web.yml`

### Настройка проекта (любое количество)

Настройка подразумевает, что проект будет иметь тоже название, что и каталог с файлом wsgi.py  
Обязательное требование: наличие в зависимостях `gunicorn==20.1.0` и настройка проекта с env_settings.py из репозитория `robodc`
Пример строится на проекте payments и домена suoper-domain.com

1. Создаём каталог проекта командой `/root/.venv/bin/ansible-playbook /root/manage/proj.yml -e 'proj_name=payments domain=super-domain.com'`  
   Произойдёт создание каталога payments, в каталог скопируются файл настройки для проекта, структура для создания образа контейнеров
2. Переходим в каталог проекта `cd payments`, в каталоге размещается фалй project.yml и каталог img
3. В каталог img/django копируем проект. Как копировался проект для репозитория `robodc`
4. В каталоге проекта `/root/payments` выполняем команду `/root/.venv/bin/ansible-playbook /root/manage/db.yml -e @project.yml --tags create`  
   Произойдёт создание БД и пользователя для неё.
5. Запускаем проект командой `/root/.venv/bin/ansible-playbook /root/manage/web.yml -e @project.yml`  
   Произойдёт копирование статики, сборка образа контейнера, запуск приложения и запуск nginx который управляет статикой.

## Управление проекта

лог запросов web: `docker logs payments_ngx`  
лог работы приложения: `docker logs payments_app`  
остановка проекта: `docker stop payments_ngx payments_app`  
удаление проекта: `docker rm payments_ngx payments_app`  
остановка и удаление `docker rm -f payments_ngx payments_app`
удаление образа проекта `docker image rm payments:latest; docker image prune -f` может быть выполнено после удаления контейнеров

## Архитектура

При созданиии проекта создаётся файл project.yml, который содержит все необходимые данные для работы проекта. Управление проекта осуществляется только при нахождении в каталоге проекта. Образ собирается с тэгом из файла project.yml. Рабочим каталогом для образа является /www/<имя проекта>, статика копируется из каталога проекта в рабочий каталог образа.
