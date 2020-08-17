FROM php:7.4-fpm-buster
MAINTAINER Oussama Chebbi <choussamaster@gmail.com>

ENV TZ=Europe/Paris \
    PROJECT_DIR=/var/www/html \
    COMPOSER_HOME=/var/www/.composer/

# Configure Node.js version
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash

RUN apt-get update && apt-get install -y \
  cron \
  git \
  gzip \
  libbz2-dev \
  libfreetype6-dev \
  libicu-dev \
  libjpeg62-turbo-dev \
  libmcrypt-dev \
  libonig-dev \
  libpng-dev \
  libsodium-dev \
  libssh2-1-dev \
  libxslt1-dev \
  libzip-dev \
  lsof \
  default-mysql-client \
  nodejs \
  vim \
  zip

RUN docker-php-ext-configure gd --with-freetype --with-jpeg

RUN docker-php-ext-install \
  bcmath \
  bz2 \
  calendar \
  exif \
  gd \
  gettext \
  intl \
  mbstring \
  mysqli \
  opcache \
  pcntl \
  pdo_mysql \
  soap \
  sockets \
  sodium \
  sysvmsg \
  sysvsem \
  sysvshm \
  xmlrpc \
  xsl \
  zip


RUN pecl channel-update pecl.php.net

## Replace next lines with below commented out version once issue is resolved
# https://github.com/php/pecl-networking-ssh2/pull/36
# https://bugs.php.net/bug.php?id=78560
#RUN curl -o /tmp/ssh2-1.2.tgz https://pecl.php.net/get/ssh2 \
#  && pear install /tmp/ssh2-1.2.tgz \
#  && rm /tmp/ssh2-1.2.tgz \
#  && docker-php-ext-enable ssh2

RUN pecl install ssh2-1.2 \
  && docker-php-ext-enable ssh2

RUN groupadd -g 1000 app \
 && useradd -g 1000 -u 1000 -d /var/www -s /bin/bash app

RUN apt-get install -y gnupg \
  && curl -sL https://deb.nodesource.com/setup_10.x | bash - \
  && apt-get install -y nodejs \
  && mkdir /var/www/.config /var/www/.npm \
  && chown app:app /var/www/.config /var/www/.npm \
  && npm install -g grunt-cli

# Install Grunt
#RUN mkdir /var/www/.config /var/www/.npm \
#  && chown app:app /var/www/.config /var/www/.npm \
#  && npm install -g grunt-cli

# Install MailHog
RUN curl -sSLO https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_amd64 \
  && chmod +x mhsendmail_linux_amd64 \
  && mv mhsendmail_linux_amd64 /usr/local/bin/mhsendmail

RUN curl -sS https://getcomposer.org/installer | \
  php -- --version=1.10.9 --install-dir=/usr/local/bin --filename=composer


RUN printf '* *\t* * *\tapp\t%s/usr/local/bin/php $PROJECT_DIR/update/cron.php\n' >> /etc/crontab \
  && printf '* *\t* * *\tapp\t%s/usr/local/bin/php $PROJECT_DIR/bin/magento cron:run\n' >> /etc/crontab \
  && printf '* *\t* * *\tapp\t%s/usr/local/bin/php $PROJECT_DIR/bin/magento setup:cron:run\n#\n' >> /etc/crontab


COPY conf/www.conf /usr/local/etc/php-fpm.d/
COPY conf/php.ini /usr/local/etc/php/
COPY conf/php-fpm.conf /usr/local/etc/
COPY bin/cronstart /usr/local/bin/
COPY bin/setmagento /usr/local/bin/
COPY conf/magento/auth.json $COMPOSER_HOME
RUN chown -R app:app $COMPOSER_HOME

RUN mkdir -p /etc/nginx/html $PROJECT_DIR /sock \
  && chown -R app:app /etc/nginx /var/www /usr/local/etc/php/conf.d /sock

#init empty magento project
RUN composer create-project --repository=https://repo.magento.com/ magento/project-community-edition $PROJECT_DIR/
RUN chown -R app:app $COMPOSER_HOME
COPY conf/magento/config.php $PROJECT_DIR/app/etc/
RUN $PROJECT_DIR/bin/magento setup:di:compile
#RUN $PROJECT_DIR/bin/magento setup:static-content:deploy -l en_US -t Magento/blank -f


USER app:app

VOLUME /var/www

WORKDIR $PROJECT_DIR

EXPOSE 9001
