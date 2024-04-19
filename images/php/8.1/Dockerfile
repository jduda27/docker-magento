FROM php:8.1.27-fpm-bookworm

ARG APP_ID=1000
RUN groupadd -g "$APP_ID" app \
  && useradd -g "$APP_ID" -u "$APP_ID" -d /var/www -s /bin/bash app

RUN mkdir -p /etc/nginx/html /var/www/html /sock \
  && chown -R app:app /etc/nginx /var/www /usr/local/etc/php/conf.d /sock

# Install node.js V.20
RUN curl -fsSL https://deb.nodesource.com/setup_20.x | bash -

RUN apt-get update && apt-get install -y \
    cron \
    default-mysql-client \
    git \
    gnupg \
    gzip \
    libbz2-dev \
    libfreetype6-dev \
    libicu-dev \
    libjpeg62-turbo-dev \
    libmagickwand-dev \
    libmcrypt-dev \
    libonig-dev \
    libpng-dev \
    libsodium-dev \
    libssh2-1-dev \
    libwebp-dev \
    libxslt1-dev \
    libzip-dev \
    lsof \
    mailutils \
    msmtp \
    nodejs \
    procps \
    strace \
    vim \
    zip \
    zlib1g-dev \
  && rm -rf /var/lib/apt/lists/* #These are used for apt upgrade reference, we can delete this to reduce size.

# Install the PHP extensions listed
RUN pecl channel-update pecl.php.net && pecl install \
    imagick-3.7.0 \
    redis-6.0.2 \
    ssh2-1.4.0 \
    swoole-5.1.1 \
    xdebug-3.2.2 \
  && pecl clear-cache \
  && rm -rf /tmp/pear #pecl is a php package manager. Swoole is a tool that is used for concurency and allows PHP code \
    # to run asyncronously particularly block IO functions.

# Configuring the PHP extensions in the php.ini file using a docker helper.
RUN docker-php-ext-configure \
    gd --with-freetype --with-jpeg --with-webp \
  && docker-php-ext-install \
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
    xsl \
    zip \
  && docker-php-ext-enable \
    imagick \
    redis \
    ssh2 \
    xdebug


# Blackfire is a web probing software specifically designed for PHP. They monitor at a function level performance and
# provide feedback on the configuration on a security and at a quality standpoint.
RUN version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
    && architecture=$(uname -m) \
    && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/$architecture/$version \
    && mkdir -p /tmp/blackfire \
    && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp/blackfire \
    && mv /tmp/blackfire/blackfire-*.so $(php -r "echo ini_get ('extension_dir');")/blackfire.so \
    && rm -rf /tmp/blackfire /tmp/blackfire-probe.tar.gz

# PHP SPX is a profiling tool, this allows for quick and easy deep dives into what processes are running and see a treemap
# of how the application is running.
RUN git clone --branch v0.4.15 --depth=1 https://github.com/NoiseByNorthwest/php-spx.git /usr/lib/php-spx \
    && cd /usr/lib/php-spx \
    && phpize \
    && ./configure \
    && make \
    && make install

# Composer duh...
RUN curl -sS https://getcomposer.org/installer | \
  php -- --version=2.2.18 --install-dir=/usr/local/bin --filename=composer

COPY conf/blackfire.ini $PHP_INI_DIR/conf.d/blackfire.ini
COPY conf/spx.ini $PHP_INI_DIR/conf.d/spx.ini
COPY conf/msmtprc /etc/msmtprc
COPY conf/php.ini $PHP_INI_DIR
COPY conf/php-fpm.conf /usr/local/etc/
COPY conf/www.conf /usr/local/etc/php-fpm.d/

USER app:app
VOLUME /var/www
WORKDIR /var/www/html
