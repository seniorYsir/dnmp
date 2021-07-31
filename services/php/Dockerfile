ARG PHP_VERSION
FROM ${PHP_VERSION}

ARG TZ
ARG PHP_EXTENSIONS
ARG CONTAINER_PACKAGE_URL


RUN if [ $CONTAINER_PACKAGE_URL ] ; then sed -i "s/dl-cdn.alpinelinux.org/${CONTAINER_PACKAGE_URL}/g" /etc/apk/repositories ; fi


COPY ./extensions /tmp/extensions

ADD ./add/grpc-1.27.0.tgz /tmp/extensions
ADD ./add/protobuf-3.11.4.tgz /tmp/extensions

WORKDIR /tmp/extensions

RUN apk add --no-cache autoconf \
	gcc \
	g++ \
	libc-dev \
    zlib-dev \
	re2c \
    linux-headers \
	git\
	openssh-client \
	make \
    && cd /tmp/extensions/grpc-1.27.0 \
    && phpize \
	&& ./configure --with-php-config=/usr/local/bin/php-config \
	&& make \
    && make install \
	&& docker-php-ext-enable grpc \
    && docker-php-ext-install pcntl \
    && cd /tmp/extensions/protobuf-3.11.4 \
    &&	phpize \
    && ./configure --with-php-config=/usr/local/bin/php-config \
    && make \
    && make install \
    && docker-php-ext-enable protobuf
    # && composer global require hirak/prestissimo



RUN chmod +x install.sh \
    && sh install.sh \
    && rm -rf /tmp/extensions

ADD ./extensions/install-php-extensions  /usr/local/bin/

RUN chmod uga+x /usr/local/bin/install-php-extensions


RUN docker-php-ext-install pcntl
RUN apk --no-cache add tzdata \
    && cp "/usr/share/zoneinfo/$TZ" /etc/localtime \
    && echo "$TZ" > /etc/timezone


# Fix: https://github.com/docker-library/php/issues/240
RUN apk add gnu-libiconv libstdc++ --no-cache --repository http://${CONTAINER_PACKAGE_URL}/alpine/edge/community/ --allow-untrusted
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php


# Install composer and change it's cache home
RUN curl -o /usr/bin/composer https://mirrors.aliyun.com/composer/composer.phar \
    && chmod +x /usr/bin/composer
ENV COMPOSER_HOME=/tmp/composer

# php image's www-data user uid & gid are 82, change them to 1000 (primary user)
RUN apk --no-cache add shadow && usermod -u 1000 www-data && groupmod -g 1000 www-data


WORKDIR /www
