FROM alpine:3.9

# TODO: Put the maintainer name in the image metadata
LABEL maintainer="Nilber Vittorazzi <nilber@vittorazzi.com>"

# TODO: Rename the builder environment variable to inform users about application you provide them
ENV BUILDER_VERSION 1.0

# TODO: Set labels used in OpenShift to describe the builder image
LABEL io.k8s.description="Alpine With Laravel" \
    io.k8s.display-name="Alpine With Laravel" \
    io.openshift.expose-services="8080:http" \
    io.openshift.tags="builder,php,laravel." \
    io.openshift.s2i.scripts-url="image:///usr/libexec/s2i"

# https://github.com/docker-library/php/issues/240
#RUN apk add --no-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing gnu-libiconv
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

RUN apk --no-cache update && apk --no-cache upgrade

# Install packages
RUN apk --no-cache add php7 php7-fpm php7-zip php7-json php7-openssl php7-curl \
    php7-zlib php7-xml php7-phar php7-intl php7-dom php7-xmlreader php7-xmlwriter php7-ctype \
    php7-mbstring php7-gd php7-session php7-pdo php7-pdo_mysql php7-tokenizer php7-posix \
    php7-fileinfo php7-opcache php7-cli php7-mcrypt php7-pcntl php7-iconv php7-simplexml \
    nginx curl git supervisor
RUN apk add --no-cache --virtual .phpize-deps $PHPIZE_DEPS imagemagick-dev libtool \
    && export CFLAGS="$PHP_CFLAGS" CPPFLAGS="$PHP_CPPFLAGS" LDFLAGS="$PHP_LDFLAGS" \
    && pecl install imagick-3.4.3 \
    && docker-php-ext-enable imagick \
    && apk add --no-cache --virtual .imagick-runtime-deps imagemagick \
    && apk del .phpize-deps

# We have to pull mongodb from the edge repository for now
#RUN apk add --no-cache --repository http://dl-3.alpinelinux.org/alpine/edge/testing php7-mongodb

# Configure nginx
COPY config/nginx.conf /etc/nginx/nginx.conf

# Configure PHP-FPM
COPY config/fpm-pool.conf /etc/php7/php-fpm.d/zzz_custom.conf
COPY config/php.ini /etc/php7/conf.d/zzz_custom.ini

# Configure supervisord
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

WORKDIR /var/www/html

#NEW
#COPY code/ /var/www/html/

RUN addgroup -S 1001 && adduser -S 1001 -G 1001

RUN mkdir -p /.composer && chmod 777 /var/www/html -R \
    && chown -R 1001:1001 /var/www/html /usr/local/bin /.composer /var/lib/nginx /var/log/nginx /var/tmp/nginx/

RUN mkdir -p /usr/libexec/
COPY ./s2i/bin/ /usr/libexec/s2i


#CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

EXPOSE 8080

USER 1001

# Configure composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer global require hirak/prestissimo
