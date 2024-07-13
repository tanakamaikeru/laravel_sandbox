FROM php:8.3-apache

RUN <<EOF
    apt-get update
    apt-get install -y --no-install-recommends \
        curl \
        g++ \
        git \
        libbz2-dev \
        libfreetype6-dev \
        libicu-dev \
        libjpeg-dev \
        libmcrypt-dev \
        libpng-dev \
        libreadline-dev \
        unzip \
        zip \
        lynx \
        sudo \
        nodejs npm
    rm -rf /var/lib/apt/lists/*
EOF


ENV APACHE_DOCUMENT_ROOT=/var/www/html/public
RUN <<EOF
    echo "ServerName laravel-apache" >> /etc/apache2/apache2.conf
    sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
    sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf
    echo "export APACHE_LYNX='lynx -dump'" >> /etc/apache2/envvars
EOF


COPY ./php.ini /usr/local/etc/php/php.ini
COPY ./server-status.conf /etc/apache2/conf-available/server-status.conf

RUN <<EOF
    pecl install xdebug
    docker-php-ext-enable xdebug
EOF
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

RUN docker-php-ext-install \
    bcmath \
    intl \
    opcache \
    pdo_mysql

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

ARG UID
ARG GID

RUN <<EOF
    groupadd -g ${GID:-1000} vscode
    useradd -m -s /bin/bash -u ${UID:-1000} -g $GID -G sudo vscode
    echo vscode:password | chpasswd
    echo "vscode ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
EOF

RUN mkdir -p /home/vscode/.composer && \
    chown -R vscode:vscode /home/vscode

RUN a2enmod rewrite

USER vscode