FROM phusion/baseimage
MAINTAINER 4 All Digital  "joe@4alldigital.com"

# Required to run some apt-get commands (fix for Unable to locate package php5 issue)
ENV DEBIAN_FRONTEND noninteractive

# Ensure UTF-8
RUN locale-gen en_GB.UTF-8
ENV LANG       en_GB.UTF-8
ENV LC_ALL     en_GB.UTF-8

ENV XHPROF_VERSION 0.9.4
ENV XDEBUG_VERSION 2.4.0
ENV XDEBUG_MD5 a9bc9c6b9e8bc913fb1f7c6f6d19f6222d430414

RUN apt-get -y update && apt-get install -y software-properties-common python-software-properties && \
    add-apt-repository ppa:ondrej/php && \
    apt-get update -y

RUN apt-get -q install -y --no-install-recommends php5.6-fpm php5.6-curl curl\
    build-essential php5.6-dev php5.6-xdebug wget php5.6-mcrypt php5.6-gd imagemagick\
    php5.6-xsl php5.6-imap php5.6-pgsql php5.6-mysql mysql-client ssmtp\
    php5.6-common php-pear nginx-extras php5.6-redis php5.6-json php5.6-cli supervisor\
    msmtp mailutils libncurses-dev zlib1g-dev php5.6-json openssh-server make\
    libreadline-dev libncurses5-dev libpcre3-dev libssl-dev perl libxslt1-dev libgd2-xpm-dev\
    libgeoip-dev libxrender1 libssl-dev libbz2-dev libcurl4-openssl-dev\
    libfreetype6 libfreetype6-dev libfontconfig libfontconfig1 libexpat-dev git && \
    apt-get autoremove

# Install XHPROF
RUN curl -L -o /root/xhprof.tgz https://pecl.php.net/get/xhprof-$XHPROF_VERSION.tgz && \
    cd /root && \
    mkdir -p /usr/src/xhprof && \
    tar -zxvf xhprof.tgz -C /usr/src/xhprof --strip-components=1 && \
    cd /usr/src/xhprof/extension && \
    phpize && \
    ./configure --with-php-config=/usr/bin/php-config && \
    make  && \
    make install && \
    cd /root && \
    rm -fr /root/xhprof-$XHPROF_VERSION.tgz && \
# Install XDEBUG
    set -x \
    && curl -SL "http://xdebug.org/files/xdebug-$XDEBUG_VERSION.tgz" -o xdebug.tgz \
    && echo "$XDEBUG_MD5  xdebug.tgz" | shasum -c - \
    && mkdir -p /usr/src/xdebug \
    && tar -xf xdebug.tgz -C /usr/src/xdebug --strip-components=1 \
    && rm xdebug.* \
    && cd /usr/src/xdebug \
    && phpize \
    && ./configure \
    && make -j"$(nproc)" \
    && make install \
    && make clean && \
# Install PHPREDIS
    wget https://github.com/phpredis/phpredis/archive/develop.tar.gz && tar -xvf develop.tar.gz && \
    cd phpredis-develop &&\
    phpize &&\
    ./configure &&\
    make && make install && \
# Install Composer
    cd $HOME && \
    curl -sS https://getcomposer.org/installer | php && \
    chmod +x composer.phar && \
    mv composer.phar /usr/local/bin/composer && \
    /usr/local/bin/composer self-update && \
# Cleanup build
    apt-get autoremove -y

RUN echo "error_reporting = E_ALL\ndisplay_startup_errors = 1\ndisplay_errors = 1" >> /etc/php/5.6/fpm/conf.d/01-errors.ini && \
    echo "error_reporting = E_ALL\ndisplay_startup_errors = 1\ndisplay_errors = 1" >> /etc/php/5.6/cli/conf.d/01-errors.ini && \
    echo "mailhub=mailcatcher:1025\nUseTLS=NO\nFromLineOverride=YES" > /etc/ssmtp/ssmtp.conf

RUN mkdir -p /var/run/sshd /var/log/supervisor && \
    usermod -u 1000 www-data && \
    usermod -a -G users www-data

COPY config/supervisor/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Install useful Drush
RUN /usr/local/bin/composer global require drush/drush:8.1.2 && \
    ln -s /root/.composer/vendor/drush/drush/drush /usr/local/bin/drush

# Setup SSH for behat usage between containers
RUN echo 'root:root' | chpasswd && \
    sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    mkdir -p /root/.ssh/

RUN drush @none dl utf8mb4_convert-7.x

COPY ./ssh/config /root/.ssh/config
COPY ./ssh/dockerdev /root/.ssh/dockerdev
COPY ./ssh/dockerdev.pub /root/.ssh/dockerdev.pub
COPY ./ssh/authorized_keys /root/.ssh/authorized_keys
COPY ./config/php-fpm.conf /etc/php5/fpm/php-fpm.conf
COPY ./config/20-xdebug.ini /etc/php5/fpm/conf.d/20-xdebug.ini

# Internal Port Expose
EXPOSE 9000 22

CMD ["/usr/bin/supervisord"]
