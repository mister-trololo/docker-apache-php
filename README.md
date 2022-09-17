# docker-apache-php

Постройка сайта на основе (пока что) apache, php, mariaDB. Со временем понимания процессов думаю буду дополнять данный документ.

apache, php, mariaDB - Всё находится в отдельных контейнерах. Образы собираются по факту с нуля, готовые не берутся.

Скажу сразу, в деле докера я новичёк, так что это просто шпаргалки для себя, но может кому тоже пригодится.

### Структура

```
├── docker 
│   ├── apache
│   ├── php7.4-fpm
│   └── db
└── site (конфиги, логи, сам сайт)
    ├── apache_config (конфиги апача)
    ├── log 
    |   ├── apache (логи апача)
    |   └── php (логи php)
    ├── php_config (конфиги php)
    ├── www (какталог с сайтами)
    |    └── info.test
    └── db_files
         └── maria_db (Файлы БД)
```
Часть конфигов и докер файлы подёргал [Отсюда](https://github.com/8ctopus/apache-php-fpm-alpine).

По MariaDB взял [Отсюда](https://github.com/yobasystems/alpine-mariadb.git)

```bash
#Создадим свою сеть
docker network create web1
#Теперь собираем отдельно апач и пхп
#Апач
cd /home/<user>/docker/apache/
#Собираем образ
docker build -t test/apache_server:v1 .
#Далее собираем контейнер. Сначало просто выдернем конфиги и скопируем их в нужное нам место
docker run --name apache_web_server --rm -d test/apache_server:v1
#Забираем конфиги
docker cp apache_web_server:/etc/apache2/. /home/<user>/site/apache_config
#В conf.d кидаем конфиг своего сайта
cp /home/<user>/docker/apache/test.conf /home/<user>/site/apache_config/conf.d/test.conf
#Тормозим контейнер
docker stop apache_web_server
#Запускаем уже нормальный апач
docker run --name apache_web_server -d \
--net web1 \
-p 80:80 \
-p 443:443 \
-v /home/<user>/site/apache_config:/etc/apache2/ \
-v /home/<user>/site/log/apache:/var/log/apache2 \
-v /home/<user>/site/www:/var/www/htdocs \
test/apache_server:v1
```
Заходим на localhost или на адрес [сайта](http://info.test), html норм работает. Для php пока что рано.
Настроим пехепе

```bash
#
cd ../php7.4-fpm/
#собираем образ
docker build -t test/php_server:v1 .
#запускаем
docker run --name php_web_server --rm -d test/php_server:v1
#тянем конфиги
docker cp php_web_server:/etc/php7/. /home/<user>/site/php_config
#Тормозимся
docker stop php_web_server
#нормальный запуск
docker run --name php_web_server -d \
--net web1 \
-p 9001:9001 \
-v /home/<user>/site/php_config:/etc/php7/ \
-v /home/<user>/site/log/php:/var/log/php7/ \
-v /home/<user>/site/www:/var/www/ \
test/php_server:v1
```

Смотрим [http://info.test/phpinfo.php](http://info.test/phpinfo.php)


Теперь добавим базу данных. В данном случае это MariaDB

```bash
cd ../db
#собираем образ
docker build -t test/mariadb:v1 .
//Создаём контейнер из образа, если надо создать юзера и БД, то пишем параметры
docker run --name db_server -d \
-e MYSQL_ROOT_PASSWORD=root_pass \
-e MYSQL_DATABASE=db_name \
-e MYSQL_USER=user_name \
-e MYSQL_PASSWORD=user_pass \
-v /home/<user>/site/db_files/maria_db:/var/lib/mysql \
test/mariadb:v1
#Если нужно в БД положить дамп, то закидываем его прям в контейнер
docker cp /home/<user>/<name_dump> <containerId>:/home/<name_dump>
#Заходим в консоль контейнера
docker exec -it db_server /bin/sh
#подключаеммся к БД
mysql -u root -p
#подключаемся к БД в котороую будем заливать дамп
use <db_name>;
#заливаем
source /home/<dump_name>;
#выходим
quit;
exit;
#БД готова
```
