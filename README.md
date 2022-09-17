# docker-apache-php

Постройка сайта на основе (пока что) apache, php, mariaDB. Со временем понимания процессов думаю буду дополнять данный документ.

apache, php, mariaDB - Всё находится в отдельных контейнерах. Образы собираются по факту с нуля, готовые не берутся.

Скажу сразу, в деле докера я новичёк, так что это просто шпаргалки для себя, но может кому тоже пригодится.

### Структура

```
├── docker 
│   ├── apache
│   │   └── test_dock (конфиг где собрано всё в кучу)
│   └── php7.4-fpm
└── site (конфиги, логи, сам сайт)
    ├── apache_config (конфиги апача)
    ├── log 
    |   ├── apache (логи апача)
    |   └── php (логи php)
    ├── php_config (конфиги php)
    └── www (какталог с сайтами)
        └── info.test
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

