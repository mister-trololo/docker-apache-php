#####################################################################
# broadtech/alpine-php7-fpm
# This Dockerfile creates an inage that deploys php7-fpm
# on Alpine Linux

# Base Image
FROM alpine

LABEL "vendor"="Mister trololo"

# Upgrade existing packages in the base image
RUN apk --no-cache upgrade

RUN echo "http://dl-cdn.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories
#RUN apk add --no-cache  --repository http://dl-cdn.alpinelinux.org/alpine/edge/testing php7
RUN apk --no-cache upgrade

# Install php-fpm from packages with out caching install files
RUN apk add --no-cache php7 php7-fpm

RUN apk add --no-cache mc

RUN apk add --no-cache \
		php7-mcrypt \
		php7-json \
		php7-bz2 \
		php7-ctype \
		php7-zlib \
		php7-mysqli \
		php7-pgsql \
        # php7-cli \
        # php7-mysql \
        php7-gd \
        php7-ldap \
        php7-odbc \
        php7-pdo \
        php7-pecl-memcache \
        php7-opcache \
        php7-pear \
        php7-xml \
        php7-xmlrpc \
        php7-mbstring \
        php7-snmp \
        php7-soap \
        php7-zip

# add user www-data
# group www-data already exists
# -H don't create home directory
# -D don't assign a password
# -S create a system user
RUN adduser -H -D -S -G www-data -s /sbin/nologin www-data

#RUN sed -i 's|user = nobody|user = www-data|g' /etc/php7/php-fpm.d/www.conf
#RUN sed -i 's|group = nobody|group = www-data|g' /etc/php7/php-fpm.d/www.conf
#RUN sed -i 's|;listen.owner = nobody|listen.owner = www-data|g' /etc/php7/php-fpm.d/www.conf
#RUN sed -i 's|;listen.group = group|listen.group = www-data|g' /etc/php7/php-fpm.d/www.conf

# php log everything
RUN sed -i 's|^error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT$|error_reporting = E_ALL|g' /etc/php7/php.ini


# Copy configuration files
COPY conf/php-fpm.conf /etc/php7/
COPY conf/www.conf /etc/php7/php-fpm.d/

# Create directory for php files.
#RUN mkdir -p /var/www/localhost/htdocs/
#COPY www/index.php /var/www/localhost/htdocs/
#RUN chmod -R +x /var/www/localhost/htdocs/

# Add entrypoint scripts and make them executable
RUN mkdir /etc/docker-entrypoint.d
COPY docker-entrypoint.d/* /etc/docker-entrypoint.d/
RUN chmod u+x /etc/docker-entrypoint.d/*

COPY scripts/run_parts_entrypoint.sh /usr/sbin/
RUN chmod u+x /usr/sbin/run_parts_entrypoint.sh

# Open port to access FPM
EXPOSE 9001/tcp

#CMD ["-D","FOREGROUND"]

#CMD httpd -k start

# Srart httpd when container runs
#ENTRYPOINT ["/usr/sbin/php-fpm7"]

# Run PHP-FPM when container starts
CMD ["/usr/sbin/run_parts_entrypoint.sh"]

# -------------------------END----------------------------------------#