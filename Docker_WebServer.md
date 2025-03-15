Вот пример настройки Docker-окружения с PHP 7.4, MySQL и phpMyAdmin:

**1. Структура проекта:**
```
my-project/
├── docker/
│   └── Dockerfile
├── docker-compose.yml
└── src/               # Ваши PHP-файлы
```

**2. Dockerfile** (docker/Dockerfile):
```dockerfile
FROM php:7.4.33-apache

# Установка системных зависимостей и расширений PHP
RUN apt-get update && apt-get install -y \
    libcurl4-openssl-dev \
    && docker-php-ext-install pdo pdo_mysql mysqli curl \
    && a2enmod rewrite \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Копируем файлы сайта (опционально, если не используем volume)
# COPY src/ /var/www/html/

WORKDIR /var/www/html
```

**3. docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    build: ./docker
    ports:
      - "80:80"
    volumes:
      - ./src:/var/www/html
    networks:
      - app-network
    depends_on:
      - db

  db:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: rootpass
      MYSQL_DATABASE: mydb
      MYSQL_USER: user
      MYSQL_PASSWORD: pass
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - app-network

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    depends_on:
      - db
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      PMA_USER: root
      PMA_PASSWORD: rootpass
    ports:
      - "8080:80"
    networks:
      - app-network

volumes:
  mysql-data:

networks:
  app-network:
    driver: bridge
```

**4. Использование:**
1. Поместите ваши PHP-файлы в папку `src`
2. Запустите контейнеры:
```bash
docker-compose up -d
```

**Доступ:**
- Веб-сервер: http://localhost
- phpMyAdmin: http://localhost:8080
- Данные для подключения к БД:
  - Хост: `db`
  - Пользователь: `user` (или `root` для админ-доступа)
  - Пароль: `pass` (или `rootpass` для root)
  - Имя БД: `mydb`

**Важные заметки:**
1. Для постоянного хранения данных БД используется Docker volume
2. Все сервисы объединены в одну сеть `app-network`
3. Для изменения настроек PHP можно:
   - Создать файл `php.ini` в папке `docker`
   - Добавить в Dockerfile копирование этого файла:
     ```dockerfile
     COPY php.ini /usr/local/etc/php/conf.d/custom.ini
     ```
4. Для использования других версий MySQL или PHP измените соответствующие теги в `docker-compose.yml` и `Dockerfile`
