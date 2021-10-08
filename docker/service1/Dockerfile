FROM ubuntu:latest

ARG PHP_VERSION=7.4

RUN set -ex \
    && echo 'Acquire::http::Pipeline-Depth "0";' > /etc/apt/apt.conf.d/99fixbadproxy \
    && echo 'Acquire::http::No-Cache=True;' >> /etc/apt/apt.conf.d/99fixbadproxy \
    && echo 'Acquire::BrokenProxy=true;' >> /etc/apt/apt.conf.d/99fixbadproxy

RUN apt-get clean && apt-get update && apt-get install -y locales
RUN locale-gen en_US.UTF-8 && \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y software-properties-common

RUN LC_ALL=en_US.UTF-8 add-apt-repository -y ppa:ondrej/php && \
    apt-get update && \
    apt-get install -y gcc \
                       nginx \
                       php${PHP_VERSION} \
                       php${PHP_VERSION}-common \
                       php${PHP_VERSION}-cli \
                       php${PHP_VERSION}-cgi \
                       php${PHP_VERSION}-fpm \
                       php${PHP_VERSION}-apcu \
                       php${PHP_VERSION}-dev \
                       php${PHP_VERSION}-mysq \
                       php${PHP_VERSION}-gd \
                       php${PHP_VERSION}-gmp \
                       php${PHP_VERSION}-imap \
                       php${PHP_VERSION}-curl \
                       php${PHP_VERSION}-mcrypt \
                       php${PHP_VERSION}-mbstring \
                       php${PHP_VERSION}-soap \
                       php${PHP_VERSION}-xml \
                       php${PHP_VERSION}-curl \
                       php${PHP_VERSION}-zip \
                       unzip \
                       vim \
                       apt-utils \
                       libzip-dev \
                       supervisor

RUN ln -sfn /usr/bin/php${PHP_VERSION} /usr/bin/php
RUN update-alternatives --set php /usr/bin/php${PHP_VERSION}

RUN /usr/bin/curl -sS https://getcomposer.org/installer | /usr/bin/php -- --install-dir=/usr/bin --filename=composer --version=2.0.0

RUN curl --silent --location https://deb.nodesource.com/setup_16.x | bash -
RUN apt-get install -y nodejs

RUN mkdir -p /var/run/php

COPY nginx/nginx.conf /etc/nginx/nginx.conf

RUN sed -i -e "s/;\?daemonize\s*=\s*yes/daemonize = no/g" /etc/php/${PHP_VERSION}/fpm/php-fpm.conf

# Configure supervisord
COPY conf/supervisord.conf /etc/supervisor/conf.d/supervisord.conf


# Nginx config
RUN rm /etc/nginx/sites-enabled/default
COPY nginx/sites-available/*.conf /etc/nginx/sites-available/
RUN ln -s /etc/nginx/sites-available/*.conf /etc/nginx/sites-enabled/*.conf


# PHP config
RUN sed -i -e "s/;\?date.timezone\s*=\s*.*/date.timezone = Australia\/Sydney/g" /etc/php/${PHP_VERSION}/fpm/php.ini

WORKDIR /var/www

# Expose ports.
EXPOSE 8080

# Let supervisord start nginx & php-fpm
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]