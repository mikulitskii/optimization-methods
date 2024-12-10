# Лабораторная работа №2 со звёздочкой

## Цели работы:
- Написать “плохой” Docker compose файл, в котором есть не менее трех “bad practices” по их написанию
- Написать “хороший” Docker compose файл, в котором эти плохие практики исправлены
- В Readme описать каждую из плохих практик в плохом файле, почему она плохая и как в хорошем она была исправлена, как исправление повлияло на результат
- После предыдущих пунктов в хорошем файле настроить сервисы так, чтобы контейнеры в рамках этого compose-проекта так же поднимались вместе, но не "видели" друг друга по сети. В отчете описать, как этого добились и кратко объяснить принцип такой изоляции


## Ход работы

### “Плохой” Docker compose файл:

```bash
version: "3.9"
services:
  web:
    image: nginx:latest
    container_name: hardcoded_web_container
    ports:
      - "80:80"
    networks:
      - default
    environment:
      - ENV=production
      - APP_SECRET_KEY=qwerty

  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: hardcoded_app_container
    privileged: true
    volumes:
      - ./app:/usr/src/app
    depends_on:
      - web
    restart: always
    networks:
      - default

  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3306:3306"
    networks:
      - default
``` 

### “Хороший” Docker compose файл:

```bash
version: "3.9"

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    environment:
      ENV: production
      APP_SECRET_KEY: /run/secrets/app_secret_key
    networks:
      - web_net
    secrets:
      - app_secret_key

  app:
    build:
      context: .
      dockerfile: Dockerfile
    volumes:
      - ./app:/usr/src/app
    restart: always
    networks:
      - app_net

  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: /run/secrets/mysql_root_password
    ports:
      - "3306:3306"
    secrets:
      - mysql_root_password
    networks:
      - db_net

networks:
  web_net:
  app_net:
  db_net:

secrets:
  mysql_root_password:
    file: ./secrets/mysql_root_password.txt
  app_secret_key:
    file: ./secrets/app_secret_key.txt
``` 


### Плохие практики “Плохом” Docker compose файле:

1. Жестко заданные имена контейнеров - Это мешает развертывать несколько копий проекта. В хорошей версии убрали `container_name`. Docker автоматически теперь генерирует уникальные имена.
2. Хранение секретов в открытом виде в файле - Переменная APP_SECRET_KEY содержит секретный ключ. В хорошем файле мы начала использовать Docker Secrets для хранения паролей.
3. Использование privileged: true без необходимости - Это дает root-доступ к хосту. В хорошей версии файла убрали `privileged`. Безопасность улучшена.
4. Пароль базы данных `MYSQL_ROOT_PASSWORD` прост и хранится в открытом виде - Это небезопасно, не надо так делать. В хорошей версии файла мы начали использовать Docker Secrets.

### Реализация изоляции:
Использование отдельных сетей web_net, db_net и app_net обеспечивает изоляцию. Контейнеры web, app и db не видят друг друга, так как они находятся в разных сетях.

## Итоги работы:
Научились создавать хорошие Docker compose файлы и стали на шаг ближе к DevOps.