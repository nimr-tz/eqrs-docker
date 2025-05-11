FROM php:8-fpm-alpine

ARG UID
ARG GID

ENV UID=${UID}
ENV GID=${GID}

RUN apk --no-cache add \
        imagemagick \
        imagemagick-dev \
        libtool \
        autoconf \
        g++ \
        make \
        pcre-dev \
        freetype-dev \
        libjpeg-turbo-dev \
        libpng-dev \
    && rm -rf /var/cache/apk/*

RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd

RUN mkdir -p /var/www/html

WORKDIR /var/www/html

COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

# Remove unused groups
RUN delgroup dialout

# Create a non-root user and group
RUN addgroup -g ${GID} --system reims \
    && adduser -G reims --system -D -s /bin/sh -u ${UID} reims

# Configure PHP-FPM to use the reims user and group
RUN sed -i "s/user = www-data/user = reims/g" /usr/local/etc/php-fpm.d/www.conf \
    && sed -i "s/group = www-data/group = reims/g" /usr/local/etc/php-fpm.d/www.conf \
    && echo "php_admin_flag[log_errors] = on" >> /usr/local/etc/php-fpm.d/www.conf \
    && echo "php_admin_value[error_log] = /var/log/php-fpm/error.log" >> /usr/local/etc/php-fpm.d/www.conf

# Install required PHP extensions
RUN docker-php-ext-install pdo pdo_mysql bcmath

# Install Imagick extension
RUN pecl install imagick \
    && docker-php-ext-enable imagick

# Install Redis extension
RUN mkdir -p /usr/src/php/ext/redis \
    && curl -L https://github.com/phpredis/phpredis/archive/5.3.4.tar.gz | tar xvz -C /usr/src/php/ext/redis --strip 1 \
    && echo 'redis' >> /usr/src/php-available-exts \
    && docker-php-ext-install redis

USER reims

CMD ["php-fpm", "-y", "/usr/local/etc/php-fpm.conf", "-R"]
