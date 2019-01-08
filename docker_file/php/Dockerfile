ARG PHP_VERSION=${PHP_VERSION}
FROM php:${PHP_VERSION}-fpm
LABEL maintainer="pota <wrpota@gmail.com>"

ENV TZ=${WORKSPACE_TIMEZONE}
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

#####################################
# xDebug Redis:
#####################################
ARG INSTALL_XDEBUG=${INSTALL_XDEBUG}
ARG INSTALL_PHPREDIS=${INSTALL_PHPREDIS}
RUN if [ ${INSTALL_XDEBUG} = true ] ; then \
        # Install the xdebug extension
        pecl install xdebug && \
        docker-php-ext-enable xdebug \
    ;fi && \
    if [ ${INSTALL_PHPREDIS} = true ] ; then \
        # Install the redis extension
        pecl install redis && \
        docker-php-ext-enable redis && \
        rm -rf /tmp/pear \
    ;fi && \
    rm -rf /var/lib/apt/lists/*

COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

EXPOSE 9000

CMD ["php-fpm", "-g", "daemon off;"]