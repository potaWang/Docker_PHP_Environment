ARG PHP_VERSION
FROM laradock/workspace:2.5-${PHP_VERSION}

LABEL maintainer="pota <wrpota@gmail.com>"

ARG PHP_VERSION

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

# Start as root
USER root

ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

# always run apt update when start and after add new source list, then clean up at end.
RUN set -xe; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    groupadd -g ${PGID} pota && \
    useradd -u ${PUID} -g pota -m pota -G docker_env && \
    usermod -p "*" pota -s /bin/bash && \
    apt-get install -yqq \
      apt-utils \
      #
      #--------------------------------------------------------------------------
      # Mandatory Software's Installation
      #--------------------------------------------------------------------------
      #
      # Mandatory Software's such as ("php-cli", "git", "vim", ....) are
      # installed on the base image 'laradock/workspace' image. If you want
      # to add more Software's or remove existing one, you need to edit the
      # base image (https://github.com/Laradock/workspace).
      #
      # next lines are here becase there is no auto build on dockerhub see https://github.com/laradock/laradock/pull/1903#issuecomment-463142846
      libzip-dev zip unzip \
      # Install the zip extension
      php${PHP_VERSION}-zip \
      # nasm
      nasm && \
      php -m | grep -q 'zip'

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# Optional Software's will only be installed if you set them to `true`
# in the `docker-compose.yml` before the build.
# Example:
#   - INSTALL_NODE=false
#   - ...
#

###########################################################################
# Set Timezone
###########################################################################

ENV TZ=${WORKSPACE_TIMEZONE}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# User Aliases
###########################################################################

USER root

COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/pota/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/pota/aliases.sh && \
    chown pota:pota /home/pota/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	  echo "" >> ~/.bashrc

USER pota

RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	  echo "" >> ~/.bashrc

###########################################################################
# Composer:
###########################################################################

USER root

# Add the composer.json
COPY ./composer.json /home/pota/.composer/composer.json

# Add the auth.json for magento 2 credentials
COPY ./auth.json /home/pota/.composer/auth.json

# Make sure that ~/.composer belongs to pota
RUN chown -R pota:pota /home/pota/.composer

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="$HOME/.composer/vendor/bin:$PATH"' >> ~/.bashrc

USER pota

# Check if global install need to be ran
ARG COMPOSER_GLOBAL_INSTALL=false
ENV COMPOSER_GLOBAL_INSTALL ${COMPOSER_GLOBAL_INSTALL}

RUN if [ ${COMPOSER_GLOBAL_INSTALL} = true ]; then \
    # run the install
    composer global install \
;fi

# Check if auth file is disabled
ARG COMPOSER_AUTH=false
ENV COMPOSER_AUTH ${COMPOSER_AUTH}

RUN if [ ${COMPOSER_AUTH} = false ]; then \
    # remove the file
    rm /home/pota/.composer/auth.json \
;fi

ARG COMPOSER_REPO_PACKAGIST
ENV COMPOSER_REPO_PACKAGIST ${COMPOSER_REPO_PACKAGIST}

RUN if [ ${COMPOSER_REPO_PACKAGIST} ]; then \
    composer config -g repo.packagist composer ${COMPOSER_REPO_PACKAGIST} \
;fi

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc

###########################################################################
# Non-root user : PHPUnit path
###########################################################################

# add ./vendor/bin to non-root user's bashrc (needed for phpunit)
USER pota

RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="/var/www/vendor/bin:$PATH"' >> ~/.bashrc

###########################################################################
# Crontab
###########################################################################

USER root

COPY ./crontab /etc/cron.d

RUN chmod -R 644 /etc/cron.d

###########################################################################
# SSH2:
###########################################################################

USER root

ARG INSTALL_SSH2=false

RUN if [ ${INSTALL_SSH2} = true ]; then \
  # Install the PHP SSH2 extension
  apt-get -y install libssh2-1-dev php${PHP_VERSION}-ssh2 \
;fi

###########################################################################
# GMP:
###########################################################################

USER root

ARG INSTALL_GMP=false
ARG PHP_VERSION=${PHP_VERSION}

RUN if [ ${INSTALL_GMP} = true ]; then \
  # Install the PHP GMP extension
  apt-get -y install php${PHP_VERSION}-gmp \
;fi

###########################################################################
# SOAP:
###########################################################################

USER root

ARG INSTALL_SOAP=false

RUN if [ ${INSTALL_SOAP} = true ]; then \
  # Install the PHP SOAP extension
  apt-get -y install libxml2-dev php${PHP_VERSION}-soap \
;fi

###########################################################################
# XSL:
###########################################################################

USER root

ARG INSTALL_XSL=false

RUN if [ ${INSTALL_XSL} = true ]; then \
  # Install the PHP XSL extension
  apt-get -y install libxslt-dev php${PHP_VERSION}-xsl \
;fi


###########################################################################
# LDAP:
###########################################################################

ARG INSTALL_LDAP=false

RUN if [ ${INSTALL_LDAP} = true ]; then \
    apt-get install -y libldap2-dev && \
    apt-get install -y php${PHP_VERSION}-ldap \
;fi

###########################################################################
# IMAP:
###########################################################################

ARG INSTALL_IMAP=false

RUN if [ ${INSTALL_IMAP} = true ]; then \
    apt-get install -y php${PHP_VERSION}-imap \
;fi

###########################################################################
# Subversion:
###########################################################################

USER root

ARG INSTALL_SUBVERSION=false

RUN if [ ${INSTALL_SUBVERSION} = true ]; then \
    apt-get install -y subversion \
;fi


###########################################################################
# xDebug:
###########################################################################

USER root

ARG INSTALL_XDEBUG=false

RUN if [ ${INSTALL_XDEBUG} = true ]; then \
    # Load the xdebug extension only with phpunit commands
    apt-get install -y php${PHP_VERSION}-xdebug && \
    sed -i 's/^;//g' /etc/php/${PHP_VERSION}/cli/conf.d/20-xdebug.ini \
;fi

# ADD for REMOTE debugging
COPY ./xdebug.ini /etc/php/${PHP_VERSION}/cli/conf.d/xdebug.ini

RUN sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /etc/php/${PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /etc/php/${PHP_VERSION}/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /etc/php/${PHP_VERSION}/cli/conf.d/xdebug.ini

###########################################################################
# Phpdbg:
###########################################################################

USER root

ARG INSTALL_PHPDBG=false

RUN if [ ${INSTALL_PHPDBG} = true ]; then \
    # Load the xdebug extension only with phpunit commands
    apt-get install -y --force-yes php${PHP_VERSION}-phpdbg \
;fi

###########################################################################
# Blackfire:
###########################################################################

ARG INSTALL_BLACKFIRE=false
ARG BLACKFIRE_CLIENT_ID
ENV BLACKFIRE_CLIENT_ID ${BLACKFIRE_CLIENT_ID}
ARG BLACKFIRE_CLIENT_TOKEN
ENV BLACKFIRE_CLIENT_TOKEN ${BLACKFIRE_CLIENT_TOKEN}

RUN if [ ${INSTALL_XDEBUG} = false -a ${INSTALL_BLACKFIRE} = true ]; then \
    curl -L https://packages.blackfire.io/gpg.key | apt-key add - && \
    echo "deb http://packages.blackfire.io/debian any main" | tee /etc/apt/sources.list.d/blackfire.list && \
    apt-get update -yqq && \
    apt-get install blackfire-agent \
;fi

###########################################################################
# ssh:
###########################################################################

ARG INSTALL_WORKSPACE_SSH=false

COPY insecure_id_rsa /tmp/id_rsa
COPY insecure_id_rsa.pub /tmp/id_rsa.pub

RUN if [ ${INSTALL_WORKSPACE_SSH} = true ]; then \
    rm -f /etc/service/sshd/down && \
    cat /tmp/id_rsa.pub >> /root/.ssh/authorized_keys \
        && cat /tmp/id_rsa.pub >> /root/.ssh/id_rsa.pub \
        && cat /tmp/id_rsa >> /root/.ssh/id_rsa \
        && rm -f /tmp/id_rsa* \
        && chmod 644 /root/.ssh/authorized_keys /root/.ssh/id_rsa.pub \
    && chmod 400 /root/.ssh/id_rsa \
    && cp -rf /root/.ssh /home/pota \
    && chown -R pota:pota /home/pota/.ssh \
;fi

###########################################################################
# MongoDB:
###########################################################################

ARG INSTALL_MONGO=false

RUN if [ ${INSTALL_MONGO} = true ]; then \
    # Install the mongodb extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      pecl install mongo && \
      echo "extension=mongo.so" >> /etc/php/${PHP_VERSION}/mods-available/mongo.ini && \
      ln -s /etc/php/${PHP_VERSION}/mods-available/mongo.ini /etc/php/${PHP_VERSION}/cli/conf.d/30-mongo.ini \
    ;fi && \
    pecl install mongodb && \
    echo "extension=mongodb.so" >> /etc/php/${PHP_VERSION}/mods-available/mongodb.ini && \
    ln -s /etc/php/${PHP_VERSION}/mods-available/mongodb.ini /etc/php/${PHP_VERSION}/cli/conf.d/30-mongodb.ini \
;fi

###########################################################################
# AMQP:
###########################################################################

ARG INSTALL_AMQP=false

RUN if [ ${INSTALL_AMQP} = true ]; then \
    apt-get install librabbitmq-dev -y && \
    pecl -q install amqp && \
    echo "extension=amqp.so" >> /etc/php/${PHP_VERSION}/mods-available/amqp.ini && \
    ln -s /etc/php/${PHP_VERSION}/mods-available/amqp.ini /etc/php/${PHP_VERSION}/cli/conf.d/30-amqp.ini \
;fi

###########################################################################
# CASSANDRA:
###########################################################################

ARG INSTALL_CASSANDRA=false

RUN if [ ${INSTALL_CASSANDRA} = true ]; then \
    apt-get install libgmp-dev -y && \
    curl https://downloads.datastax.com/cpp-driver/ubuntu/18.04/dependencies/libuv/v1.28.0/libuv1-dev_1.28.0-1_amd64.deb -o libuv1-dev.deb && \
    curl https://downloads.datastax.com/cpp-driver/ubuntu/18.04/dependencies/libuv/v1.28.0/libuv1_1.28.0-1_amd64.deb -o libuv1.deb && \
    curl https://downloads.datastax.com/cpp-driver/ubuntu/18.04/cassandra/v2.12.0/cassandra-cpp-driver-dev_2.12.0-1_amd64.deb -o cassandra-cpp-driver-dev.deb && \
    curl https://downloads.datastax.com/cpp-driver/ubuntu/18.04/cassandra/v2.12.0/cassandra-cpp-driver_2.12.0-1_amd64.deb -o cassandra-cpp-driver.deb && \
    dpkg -i libuv1.deb && \
    dpkg -i libuv1-dev.deb && \
    dpkg -i cassandra-cpp-driver.deb && \
    dpkg -i cassandra-cpp-driver-dev.deb && \
    rm libuv1.deb libuv1-dev.deb cassandra-cpp-driver-dev.deb cassandra-cpp-driver.deb && \
    cd /usr/src && \
    git clone https://github.com/datastax/php-driver.git && \
    cd /usr/src/php-driver/ext && \
    phpize && \
    mkdir /usr/src/php-driver/build && \
    cd /usr/src/php-driver/build && \
    ../ext/configure > /dev/null && \
    make clean >/dev/null && \
    make >/dev/null 2>&1 && \
    make install && \
    echo "extension=cassandra.so" >> /etc/php/${PHP_VERSION}/mods-available/cassandra.ini && \
    ln -s /etc/php/${PHP_VERSION}/mods-available/cassandra.ini /etc/php/${PHP_VERSION}/cli/conf.d/30-cassandra.ini \
;fi

###########################################################################
# PHP REDIS EXTENSION
###########################################################################

ARG INSTALL_PHPREDIS=false

RUN if [ ${INSTALL_PHPREDIS} = true ]; then \
    apt-get update -yqq && \
    apt-get install -yqq php-redis \
;fi

###########################################################################
# Swoole EXTENSION
###########################################################################

ARG INSTALL_SWOOLE=false

RUN if [ ${INSTALL_SWOOLE} = true ]; then \
    # Install Php Swoole Extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      pecl -q install swoole-2.0.10; \
    else \
      if [ $(php -r "echo PHP_MINOR_VERSION;") = "0" ]; then \
        pecl install swoole-2.2.0; \
      else \
        pecl install swoole; \
      fi \
    fi && \
    echo "extension=swoole.so" >> /etc/php/${PHP_VERSION}/mods-available/swoole.ini && \
    ln -s /etc/php/${PHP_VERSION}/mods-available/swoole.ini /etc/php/${PHP_VERSION}/cli/conf.d/20-swoole.ini \
    && php -m | grep -q 'swoole' \
;fi

###########################################################################
# Taint EXTENSION
###########################################################################

ARG INSTALL_TAINT=false

RUN if [ "${INSTALL_TAINT}" = true ]; then \
    # Install Php TAINT Extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "7" ]; then \
      pecl install taint && \
      echo "extension=taint.so" >> /etc/php/${PHP_VERSION}/mods-available/taint.ini && \
      ln -s /etc/php/${PHP_VERSION}/mods-available/taint.ini /etc/php/${PHP_VERSION}/cli/conf.d/20-taint.ini && \
      php -m | grep -q 'taint'; \
    fi \
;fi

###########################################################################
# Libpng16 EXTENSION
###########################################################################

ARG INSTALL_LIBPNG=false

RUN if [ ${INSTALL_LIBPNG} = true ]; then \
    apt-get update && \
    apt-get install libpng16-16 \
;fi

###########################################################################
# Inotify EXTENSION:
###########################################################################

ARG INSTALL_INOTIFY=false

RUN if [ ${INSTALL_INOTIFY} = true ]; then \
    pecl -q install inotify && \
    echo "extension=inotify.so" >> /etc/php/${PHP_VERSION}/mods-available/inotify.ini && \
    ln -s /etc/php/${PHP_VERSION}/mods-available/inotify.ini /etc/php/${PHP_VERSION}/cli/conf.d/20-inotify.ini \
;fi

###########################################################################
# fswatch
###########################################################################

ARG INSTALL_FSWATCH=false

RUN if [ ${INSTALL_FSWATCH} = true ]; then \
    apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 47FE03C1 \
    && add-apt-repository -y ppa:hadret/fswatch \
    || apt-get update -yqq \
    && apt-get -y install fswatch \
;fi

###########################################################################
# IonCube Loader
###########################################################################

ARG INSTALL_IONCUBE=false

RUN if [ ${INSTALL_IONCUBE} = true ]; then \
    # Install the php ioncube loader
    curl -L -o /tmp/ioncube_loaders_lin_x86-64.tar.gz https://downloads.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz \
    && tar zxpf /tmp/ioncube_loaders_lin_x86-64.tar.gz -C /tmp \
    && mv /tmp/ioncube/ioncube_loader_lin_${PHP_VERSION}.so $(php -r "echo ini_get('extension_dir');")/ioncube_loader.so \
    && echo "zend_extension=ioncube_loader.so" >> /etc/php/${PHP_VERSION}/mods-available/0ioncube.ini \
    && rm -rf /tmp/ioncube* \
;fi

###########################################################################
# PHP Aerospike:
###########################################################################

USER root

ARG INSTALL_AEROSPIKE=false

RUN set -xe; \
  if [ ${INSTALL_AEROSPIKE} = true ]; then \
    # Fix dependencies for PHPUnit within aerospike extension
    apt-get -y install sudo wget && \
    # Install the php aerospike extension
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      curl -L -o /tmp/aerospike-client-php.tar.gz https://github.com/aerospike/aerospike-client-php5/archive/master.tar.gz; \
    else \
      curl -L -o /tmp/aerospike-client-php.tar.gz https://github.com/aerospike/aerospike-client-php/archive/master.tar.gz; \
    fi \
    && mkdir -p /tmp/aerospike-client-php \
    && tar -C /tmp/aerospike-client-php -zxvf /tmp/aerospike-client-php.tar.gz --strip 1 \
    && \
      if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
        ( \
            cd /tmp/aerospike-client-php/src/aerospike \
            && phpize \
            && ./build.sh \
            && make install \
        ) \
      else \
        ( \
            cd /tmp/aerospike-client-php/src \
            && phpize \
            && ./build.sh \
            && make install \
        ) \
      fi \
    && rm /tmp/aerospike-client-php.tar.gz \
    && echo 'extension=aerospike.so' >> /etc/php/${PHP_VERSION}/cli/conf.d/aerospike.ini \
    && echo 'aerospike.udf.lua_system_path=/usr/local/aerospike/lua' >> /etc/php/${PHP_VERSION}/cli/conf.d/aerospike.ini \
    && echo 'aerospike.udf.lua_user_path=/usr/local/aerospike/usr-lua' >> /etc/php/${PHP_VERSION}/cli/conf.d/aerospike.ini \
  ;fi

###########################################################################
# PHP V8JS:
###########################################################################

USER root

ARG INSTALL_V8JS=false

RUN set -xe; \
  if [ ${INSTALL_V8JS} = true ]; then \
    add-apt-repository -y ppa:pinepain/libv8-archived \
    && apt-get update -yqq \
    && apt-get install -y libv8-5.4 && \
    if [ $(php -r "echo PHP_MAJOR_VERSION;") = "5" ]; then \
      pecl install v8js-0.6.4; \
    else \
      pecl install v8js; \
    fi \
    && echo "extension=v8js.so" >> /etc/php/${PHP_VERSION}/cli/php.ini \
    && php -m | grep -q 'v8js' \
  ;fi

###########################################################################
# Laravel Envoy:
###########################################################################

USER pota

ARG INSTALL_LARAVEL_ENVOY=false

RUN if [ ${INSTALL_LARAVEL_ENVOY} = true ]; then \
    # Install the Laravel Envoy
    composer global require "laravel/envoy=~1.0" \
;fi

###########################################################################
# Laravel Installer:
###########################################################################

USER pota

ARG INSTALL_LARAVEL_INSTALLER=false

RUN if [ ${INSTALL_LARAVEL_INSTALLER} = true ]; then \
    # Install the Laravel Installer
	composer global require "laravel/installer" \
;fi

USER root

ARG COMPOSER_REPO_PACKAGIST
ENV COMPOSER_REPO_PACKAGIST ${COMPOSER_REPO_PACKAGIST}

RUN if [ ${COMPOSER_REPO_PACKAGIST} ]; then \
    composer config -g repo.packagist composer ${COMPOSER_REPO_PACKAGIST} \
;fi

ARG INSTALL_LARAVEL_INSTALLER=false

RUN if [ ${INSTALL_LARAVEL_INSTALLER} = true ]; then \
    # Install the Laravel Installer
	composer global require "laravel/installer" \
;fi

###########################################################################
# Deployer:
###########################################################################

USER root

ARG INSTALL_DEPLOYER=false

RUN if [ ${INSTALL_DEPLOYER} = true ]; then \
    # Install the Deployer
    # Using Phar as currently there is no support for laravel 4 from composer version
    # Waiting to be resolved on https://github.com/deployphp/deployer/issues/1552
    curl -LO https://deployer.org/deployer.phar && \
    mv deployer.phar /usr/local/bin/dep && \
    chmod +x /usr/local/bin/dep \
;fi

###########################################################################
# Symfony:
###########################################################################

USER root

ARG INSTALL_SYMFONY=false

RUN if [ ${INSTALL_SYMFONY} = true ]; then \
  mkdir -p /usr/local/bin \
  && curl -LsS https://symfony.com/installer -o /usr/local/bin/symfony \
  && chmod a+x /usr/local/bin/symfony \
  #  Symfony 3 alias
  && echo 'alias dev="php bin/console -e=dev"' >> ~/.bashrc \
  && echo 'alias prod="php bin/console -e=prod"' >> ~/.bashrc \
  #  Symfony 2 alias
  #  && echo 'alias dev="php app/console -e=dev"' >> ~/.bashrc \
  #  && echo 'alias prod="php app/console -e=prod"' >> ~/.bashrc \
;fi

###########################################################################
# PYTHON:
###########################################################################

ARG INSTALL_PYTHON=false

RUN if [ ${INSTALL_PYTHON} = true ]; then \
  apt-get -y install python python-pip python-dev build-essential  \
  && python -m pip install --upgrade pip  \
  && python -m pip install --upgrade virtualenv \
;fi

###########################################################################
# pgsql client
###########################################################################

USER root

ARG INSTALL_PG_CLIENT=false

RUN if [ ${INSTALL_PG_CLIENT} = true ]; then \
    # Install the pgsql client
    apt-get install wget \
    && add-apt-repository "deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main" \
    && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
    && apt-get update \
    && apt-get -y install postgresql-client-10 \
;fi

###########################################################################
# MySQL Client:
###########################################################################

USER root

ARG INSTALL_MYSQL_CLIENT=false

RUN if [ ${INSTALL_MYSQL_CLIENT} = true ]; then \
    apt-get update -yqq && \
    apt-get -y install mysql-client \
;fi

###########################################################################
# ping:
###########################################################################

USER root

ARG INSTALL_PING=false

RUN if [ ${INSTALL_PING} = true ]; then \
    apt-get update -yqq && \
    apt-get -y install inetutils-ping \
;fi

###########################################################################
# sshpass:
###########################################################################

USER root

ARG INSTALL_SSHPASS=false

RUN if [ ${INSTALL_SSHPASS} = true ]; then \
    apt-get update -yqq && \
    apt-get -y install sshpass \
;fi

###########################################################################
# FFMpeg:
###########################################################################

USER root

ARG INSTALL_FFMPEG=false

RUN if [ ${INSTALL_FFMPEG} = true ]; then \
  apt-get -y install ffmpeg \
;fi

###########################################################################
# GNU Parallel:
###########################################################################

USER root

ARG INSTALL_GNU_PARALLEL=false

RUN if [ ${INSTALL_GNU_PARALLEL} = true ]; then \
  apt-get -y install parallel \
;fi


###########################################################################
# Check PHP version:
###########################################################################

RUN set -xe; php -v | head -n 1 | grep -q "PHP ${PHP_VERSION}."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

# Set default work directory
WORKDIR /var/www