FROM public.ecr.aws/docker/library/php:7.4-apache-buster

ENV COMPOSER_ALLOW_SUPERUSER 1
ENV DOCUMENT_ROOT /var/www/html
ENV APACHE_DOCUMENT_ROOT $DOCUMENT_ROOT/web/

RUN apt-get clean && apt-get update && apt-get install -y lsb-release
RUN echo deb http://ftp.debian.org/debian $(lsb_release -sc)-backports main > /etc/apt/sources.list.d/debian-backports.list
RUN apt-get update && apt-get upgrade -y

RUN apt-get install -y build-essential openssl g++ locales-all git wget imagemagick apt-utils rsync vim mc unzip mariadb-client gettext-base
RUN apt-get install -y --no-install-recommends pngcrush jpegoptim exiftool curl
RUN apt-get install -y libssl-dev libcurl4-openssl-dev pkg-config libbz2-dev libldb-dev libldap2-dev libxml2-dev libpng-dev libxml2-dev html2text libxslt-dev libc-client-dev libkrb5-dev libmagickwand-dev python-certbot-apache libonig-dev

RUN apt-get install -y libzip-dev

# PHP modules
RUN docker-php-ext-configure imap --with-kerberos --with-imap-ssl
RUN docker-php-ext-install bz2 curl json ldap mbstring xml xmlrpc zip mysqli bcmath intl soap zip bz2 opcache xsl pdo pdo_mysql fileinfo exif zip imap gd
RUN pecl install redis-3.1.6 imagick
RUN docker-php-ext-enable redis imagick

# Apache modules + settings
RUN a2enmod rewrite && a2enmod ssl
COPY 002-vh.conf /etc/apache2/sites-enabled
RUN rm /etc/apache2/sites-enabled/000-default.conf

# Composer
RUN wget https://getcomposer.org/download/1.10.17/composer.phar
RUN EXPECTED_CHECKSUM="6fa00eba5103ce6750f94f87af8356e12cc45d5bbb11a140533790cf60725f1c"; \
ACTUAL_CHECKSUM="$(php -r "echo hash_file('sha256', 'composer.phar');")"; \
if [ "$EXPECTED_CHECKSUM" != "$ACTUAL_CHECKSUM" ]; then \
    >&2 echo 'ERROR: Invalid installer checksum'; \
    echo $ACTUAL_CHECKSUM; \
    echo "\n"; \
    echo $EXPECTED_CHECKSUM; \
    exit 1; \
fi 

RUN mv composer.phar /usr/bin/composer
RUN chmod 755 /usr/bin/composer

RUN composer global require hirak/prestissimo

# Install NodeJS + npm
RUN apt-get install -y gnupg2
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash -

RUN apt-get install -y nodejs

RUN npm install gulp -g

# User & file permissions
RUN usermod -u 999 www-data \
    && groupmod -g 999 www-data
RUN chown -R www-data: $DOCUMENT_ROOT

CMD ["apache2-foreground"]

