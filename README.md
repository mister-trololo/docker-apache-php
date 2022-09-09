# docker-apache-php

Как то ничего не выходит

Сначала сделам всё в одной куче, то есть пхп и апач в одном контейнере

```bash
cd /home/<user>/docker/apache/test_dock
#собираем образ
docker build -t test/temp_web_server:v1 .
#Собираем контейнер
docker run --name temp_web_server --rm -d \
-p 80:80 \
-p 443:443 \
test/temp_web_server:v1
    
```
Заходим по [адресу](http://localhost/phpinfo.php)

Взято [Отсюда](https://github.com/8ctopus/apache-php-fpm-alpine) И получаем сразу результат.

```bash
#Создадим свою сеть
docker network create web1
#Тормозим контейнер
docker stop temp_web_server
#Заодно он удалился
#Теперь собираем отдельно апач и пхп
#Апач
cd ../
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
#Запускаем уже нормальный апачугу
docker run --name apache_web_server -d \
--net web1 \
-p 80:80 \
-p 443:443 \
-v /home/<user>/site/apache_config:/etc/apache2/ \
-v /home/<user>/site/log/apache:/var/log/apache2 \
-v /home/<user>/site/www:/var/www/htdocs \
test/apache_server:v1
```
Заходим на localhost или на адрес сайта, html норм работает. Для php пока что рано.
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
test/php_server:v1
```

```bash
      
    
```

```bash
      
    
```
