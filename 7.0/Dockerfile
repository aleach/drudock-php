FROM phusion/baseimage
MAINTAINER 4 All Digital  "joe@4alldigital.com"

# Required to run some apt-get commands (fix for Unable to locate package php5 issue)
ENV DEBIAN_FRONTEND noninteractive

# Ensure UTF-8
RUN locale-gen en_GB.UTF-8
ENV LANG       en_GB.UTF-8
ENV LC_ALL     en_GB.UTF-8

RUN apt-get -y update && apt-get install -y software-properties-common python-software-properties && \
    add-apt-repository ppa:ondrej/php

RUN apt-get update \
    && apt-get -q install -y --no-install-recommends \
        php7.0-fpm \
        php7.0-cli \
        php7.0-common \
        php-redis \
        php-xdebug \
        php7.0-curl \
        php7.0-mysql \
        php7.0-gd \
        php7.0-zip \
        php7.0-xml \
        php7.0-mcrypt \
        php7.0-mbstring \
        php7.0-soap \
        php7.0-json \
        php7.0-opcache \
        php7.0-bcmath \
        php7.0-pspell \
        php7.0-sqlite3 \
        php7.0-dev pkg-config php-pear && \
    # libs
    apt-get update \
    && apt-get -q install -y --no-install-recommends \
        libcurl4-openssl-dev \
        libmcrypt-dev \
        libxml2-dev \
        libjpeg-dev \
        libjpeg62 \
        libfreetype6-dev \
        libmysqlclient-dev \
        libmagickwand-dev \
        libgmp-dev \
        libicu-dev \
        libpspell-dev \
        librecode-dev \
        libxpm-dev \
        libssl-dev \
        build-essential \
        supervisor \
        imagemagick \
        ssh \
        git \
        ssmtp \
        mysql-client \
        zip unzip \
        rsync \
        php-mysql && \
     pecl install apcu-5.1.3 && \
    # Install Composer
    cd $HOME && \
        curl -sS https://getcomposer.org/installer | php && \
        chmod +x composer.phar && \
        mv composer.phar /usr/local/bin/composer && \
        /usr/local/bin/composer self-update && \
    # cleanup
    rm -fr /tmp/* /var/lib/apt/lists/* /var/tmp/* \
    && apt-get autoremove -y \
    && apt-get autoclean \
    && apt-get clean


# Install drupal console with composer
RUN curl https://drupalconsole.com/installer -L -o drupal.phar && \
    mv drupal.phar /usr/local/bin/drupal && \
    chmod +x /usr/local/bin/drupal && \
    echo "PATH=$PATH:~/.composer/vendor/bin" >> ~/.bashrc

# Install useful Drush
RUN /usr/local/bin/composer global require drush/drush:8.1.2 && \
    ln -s /root/.composer/vendor/drush/drush/drush /usr/local/bin/drush

RUN mkdir /root/.drush/

RUN cd /root/.drush/ && \
    curl -O https://raw.githubusercontent.com/previousnext/drush_cmi_tools/8.x-1.x/drush_cmi_tools.drush.inc && \
    /usr/local/bin/drush cc drush

RUN /usr/local/bin/composer global require "squizlabs/php_codesniffer=*"

RUN echo "mailhub=mailcatcher:1025\nUseTLS=NO\nFromLineOverride=YES" > /etc/ssmtp/ssmtp.conf

# Setup SSH for behat usage between containers
RUN echo 'root:root' | chpasswd && \
    sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    mkdir -p /root/.ssh/

RUN mkdir -p /app/www
WORKDIR /app/www

RUN mkdir -p /var/run/sshd /var/log/supervisor && \
    usermod -u 1000 www-data && \
    usermod -a -G users www-data

COPY ./config/supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ./config/php-fpm.conf /etc/php/7.0/fpm/php-fpm.conf
COPY ./config/www.conf /etc/php/7.0/fpm/pool.d/www.conf
COPY ./config/php.ini /etc/php/7.0/fpm/php.ini
COPY ./config/cli-php.ini /etc/php/7.0/fpm/cli-php.ini
COPY ./config/htaccess /tmp/.htaccess
COPY ./config/20-xdebug.ini /etc/php/7.0/fpm/conf.d/20-xdebug.ini

COPY ./ssh/config /root/.ssh/config
COPY ./ssh/dockerdev /root/.ssh/dockerdev
COPY ./ssh/dockerdev.pub /root/.ssh/dockerdev.pub
COPY ./ssh/authorized_keys /root/.ssh/authorized_keys


RUN mkdir -p /var/run/php/ && touch /var/run/php/php7.0-fpm.pid

# Internal Port Expose
EXPOSE 9000 22

CMD ["/usr/bin/supervisord"]
