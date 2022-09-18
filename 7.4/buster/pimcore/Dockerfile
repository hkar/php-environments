FROM public.ecr.aws/docker/library/php:7.4-apache-buster

ENV COMPOSER_ALLOW_SUPERUSER 1
ENV APACHE_DOCUMENT_ROOT /var/www/html/web/web

RUN apt-get clean && apt-get update && apt-get install -y lsb-release gnupg2

RUN apt-get update && apt-get upgrade -y

RUN apt-get install -y build-essential openssl g++ locales-all git wget imagemagick apt-utils rsync unzip ssh git gettext-base
RUN apt-get install -y --no-install-recommends ffmpeg libreoffice pngcrush jpegoptim exiftool curl
RUN apt-get install -y libssl-dev libcurl4-openssl-dev pkg-config libbz2-dev libldb-dev libldap2-dev libxml2-dev libpng-dev libxml2-dev html2text libxslt-dev libc-client-dev libkrb5-dev libmagickwand-dev python-certbot-apache libssl-dev linux-headers-amd64 libonig-dev libzip-dev

RUN docker-php-ext-configure imap --with-kerberos --with-imap-ssl
RUN docker-php-ext-install bz2 curl json ldap mbstring xml xmlrpc zip mysqli gd bcmath intl soap zip bz2 opcache xsl pdo pdo_mysql fileinfo exif zip imap 

RUN pecl install redis-3.1.6 imagick
RUN docker-php-ext-enable redis imagick

RUN a2enmod rewrite && a2enmod ssl
COPY 002-vh.conf /etc/apache2/sites-enabled
RUN rm /etc/apache2/sites-enabled/000-default.conf

RUN php -r "copy('https://getcomposer.org/download/2.0.6/composer.phar', 'composer.phar');"
RUN EXPECTED_CHECKSUM="e5e39d8fa699ddb09844f6b4388280ff2c805fef0a7c26580c5bbc91ac35091b"; \
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


RUN php -r "copy('https://kurzor-helper-files.s3-eu-west-1.amazonaws.com/nodejs/setup_12.sh', 'node_setup.sh');"
RUN php -r "copy('https://kurzor-helper-files.s3-eu-west-1.amazonaws.com/nodejs/setup_12.sh.sig', 'node_setup.sh.sig');"
RUN curl https://kurzor-helper-files.s3-eu-west-1.amazonaws.com/hkar.asc | gpg --import
RUN gpg --verify node_setup.sh.sig node_setup.sh
RUN bash node_setup.sh && rm node_setup.sh node_setup.sh.sig
RUN apt-get install -y nodejs
RUN npm install -g less
RUN npm install gulp-cli -g
RUN npm install gulp -D

RUN usermod -u 999 www-data \
    && groupmod -g 999 www-data

CMD ["apache2-foreground"]
