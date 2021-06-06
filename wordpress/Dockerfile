FROM wordpress:5-php7.4-fpm

RUN apt-get update \
  && apt-get install -y git openssl imagemagick perl libtidy-dev

ENV PHPREDIS_VERSION=5.0.2 \
    CONFIG_VAR_FLAG=WPFPM_ \
    PAGER=more \
    WP_PLUGINS="redis-cache w3-total-cache" \
    ENABLE_HYPERDB=false \
    ENABLE_CRON=false

# Install needed php extensions: brotli
#
RUN apt-get install -y libbrotli-dev \
  && git clone --recursive --depth=1 https://github.com/kjdev/php-ext-brotli.git /usr/src/php/ext/brotli \
  && cd /usr/src/php/ext/brotli \
  && phpize \
  && ./configure --with-libbrotli \
  && make \
  && make install

RUN docker-php-ext-install brotli
RUN docker-php-ext-enable brotli

# Install needed php extensions: redis
#
RUN docker-php-source extract \
  && curl -L -o /tmp/redis.tar.gz https://github.com/phpredis/phpredis/archive/$PHPREDIS_VERSION.tar.gz \
  && tar xfz /tmp/redis.tar.gz \
  && mv phpredis-$PHPREDIS_VERSION /usr/src/php/ext/redis \
  && docker-php-ext-install redis \
  && docker-php-source delete \
  && curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
  && chmod +x wp-cli.phar \
  && mv wp-cli.phar /usr/local/bin/wp \
  && rm -r /tmp/*

RUN docker-php-ext-install tidy \
  && docker-php-ext-enable tidy

RUN mkdir -p /usr/src/php/ext

# Install needed php extensions: memcached
#
RUN apt-get install -y libpq-dev libmemcached-dev libz-dev libzip-dev\
  && curl -o /tmp/memcached.tgz -SL https://pecl.php.net/get/memcached-3.1.5.tgz \
  && tar -xf /tmp/memcached.tgz -C /usr/src/php/ext/ \
  && mv /usr/src/php/ext/memcached-3.1.5 /usr/src/php/ext/memcached \
  && echo extension=memcached.so >> /usr/local/etc/php/conf.d/memcached.ini && \
        rm /tmp/memcached.tgz

RUN { \
        echo 'opcache.memory_consumption=128'; \
        echo 'opcache.interned_strings_buffer=8'; \
        echo 'opcache.max_accelerated_files=4000'; \
        echo 'opcache.revalidate_freq=2'; \
        echo 'opcache.fast_shutdown=1'; \
        echo 'opcache.enable_cli=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini

# Install needed php extensions: memcache
#
RUN apt-get install --no-install-recommends -y unzip libssl-dev libpcre3 libpcre3-dev && \
    cd /usr/src/php/ext/ && \
    curl -sSL -o php7.zip https://github.com/websupport-sk/pecl-memcache/archive/NON_BLOCKING_IO_php7.zip && \
    unzip php7.zip && \
    mv pecl-memcache-NON_BLOCKING_IO_php7 memcache && \
    docker-php-ext-configure memcache --with-php-config=/usr/local/bin/php-config && \
    docker-php-ext-install memcache && \
    echo "extension=memcache.so" > /usr/local/etc/php/conf.d/ext-memcache.ini && \
    rm -rf /tmp/pecl-memcache-php7 php7.zip


# Install needed php extensions: zip
#
RUN apt-get install -y libz-dev && \
    curl -o zip.tgz -SL http://pecl.php.net/get/zip-1.15.1.tgz && \
        tar -xf zip.tgz -C /usr/src/php/ext/ && \
        rm zip.tgz && \
        mv /usr/src/php/ext/zip-1.15.1 /usr/src/php/ext/zip

RUN docker-php-ext-install memcached
RUN docker-php-ext-install memcache
RUN docker-php-ext-install zip

# Remove defaults from WP
RUN cd /usr/src/wordpress/wp-content/plugins/ && rm -R -- */ && rm hello.php

# PHP-FPM Upload limit increase
ADD conf/php/uploads.ini /usr/local/etc/php/conf.d/

# Inherit & override default entrypoint
COPY conf/script/docker-entrypoint2.sh /usr/local/bin/

RUN chmod +x /usr/local/bin/docker-entrypoint2.sh

ENTRYPOINT ["/usr/local/bin/docker-entrypoint2.sh"]
CMD ["php-fpm"]