################################################################
# broadtech/alpine-apache2
# This Dockerfile creats an image that deploys apache2
# web server on Alpine Linux
# To deploy apache2 on alpine just run this single command
#
#     $ sudo docker run broadtech/alpine-apache2
#
# You cound use this Docker file to create your own image
# Why Alpine Linux?
# Ans: Small, Simple, Secure. https://alpinelinux.org/about/

# Base Image - Official Alpine
FROM alpine

LABEL "vendor"="BroadTech Innovations PVT LTD"

# Upgrade existing packages in the base image
RUN apk --no-cache upgrade

RUN echo "http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories
#RUN apk add --no-cache  --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing php7
# update apk repositories
RUN apk update

# upgrade all
RUN apk upgrade

#добавил MC, вдруг пригодится
RUN apk add mc

# Install apache from packages with out caching install files
#RUN apk add --no-cache apache2 apache-mod-fcgid apache2-utils apache2-ctl
RUN apk add --no-cache apache2 apache2-proxy

# add user www-data
# group www-data already exists
# -H don't create home directory
# -D don't assign a password
# -S create a system user
RUN adduser -H -D -S -G www-data -s /sbin/nologin www-data

# update user and group apache runs under
RUN sed -i 's|User apache|User www-data|g' /etc/apache2/httpd.conf
RUN sed -i 's|Group apache|Group www-data|g' /etc/apache2/httpd.conf

#Временно добавлю домен в глобальные настройки
RUN sed -i 's|#ServerName www.example.com:80|ServerName 127.0.0.1:80|g' /etc/apache2/httpd.conf

# enable mod rewrite (rewrite urls in htaccess)
RUN sed -i 's|#LoadModule rewrite_module modules/mod_rewrite.so|LoadModule rewrite_module modules/mod_rewrite.so|g' /etc/apache2/httpd.conf

# authorize all directives in .htaccess
RUN sed -i 's|    AllowOverride None|    AllowOverride All|g' /etc/apache2/httpd.conf

# authorize all changes from htaccess
RUN sed -i 's|Options Indexes FollowSymLinks|Options All|g' /etc/apache2/httpd.conf

# update error and access logs location
RUN mkdir -p /var/log/apache2
RUN sed -i 's| logs/error.log| /var/log/apache2/error_log|g' /etc/apache2/httpd.conf
RUN sed -i 's| logs/access.log| /var/log/apache2/access_log|g' /etc/apache2/httpd.conf

# switch from mpm_prefork to mpm_event
RUN sed -i 's|LoadModule mpm_prefork_module modules/mod_mpm_prefork.so|#LoadModule mpm_prefork_module modules/mod_mpm_prefork.so|g' /etc/apache2/httpd.conf
RUN sed -i 's|#LoadModule mpm_event_module modules/mod_mpm_event.so|LoadModule mpm_event_module modules/mod_mpm_event.so|g' /etc/apache2/httpd.conf

# enable important apache modules
RUN sed -i 's|#LoadModule deflate_module modules/mod_deflate.so|LoadModule deflate_module modules/mod_deflate.so|g' /etc/apache2/httpd.conf
RUN sed -i 's|#LoadModule expires_module modules/mod_expires.so|LoadModule expires_module modules/mod_expires.so|g' /etc/apache2/httpd.conf
RUN sed -i 's|#LoadModule ext_filter_module modules/mod_ext_filter.so|LoadModule ext_filter_module modules/mod_ext_filter.so|g' /etc/apache2/httpd.conf

# set working dir
WORKDIR /var/www/html/

# Open port for httpd access
# expose ports
EXPOSE 80/tcp
EXPOSE 443/tcp

VOLUME ["/etc/apache2"]

# Run httpd in foreground so that the container does not quit
# soon after start
# To run this container in the back ground use the -d option
#
#     $ sudo docker run -d broadtech/alpine-apache2
#
CMD ["-D","FOREGROUND"]

#CMD httpd -k start

# Srart httpd when container runs
ENTRYPOINT ["/usr/sbin/httpd"]

###--------------------END-------------------------------------#