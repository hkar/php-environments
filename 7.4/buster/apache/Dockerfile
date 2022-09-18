FROM public.ecr.aws/docker/library/php:7.4-apache-buster

ENV COMPOSER_ALLOW_SUPERUSER 1

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
