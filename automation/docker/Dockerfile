# ./automation/docker/Dockerfile

# Base
FROM php:8.3.7-apache-bookworm

# Define workdir
WORKDIR /var/www/html
    
# Softwares
RUN apt-get update && apt-get install -y \
    acl \
    libicu-dev \
    nano \
    wget \
    zip\
    git-all\
    locales apt-utils git libicu-dev g++ libpng-dev libxml2-dev libzip-dev libonig-dev libxslt-dev unzip libpq-dev nodejs npm wget \
        apt-transport-https lsb-release ca-certificates

# Apache modules
RUN a2enmod \
    rewrite \
    expires \
    headers

# Extension PHP
RUN docker-php-ext-install \
    pdo_mysql \
    opcache \
    intl 

# Copy php.ini
RUN cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \
    && echo 'short_open_tag = Off' >> /usr/local/etc/php/php.ini 


# Add vhost
COPY ./automation/docker/default-vhost.conf /etc/apache2/sites-available/
RUN a2dissite 000-default.conf \
    && a2ensite default-vhost.conf 

# Symfony CLI
RUN wget https://get.symfony.com/cli/installer -O - | bash \
    && chmod +x /root/.symfony5/bin/symfony \
    && mv /root/.symfony5/bin/symfony /usr/local/bin/symfony

# Composer
COPY ./automation/scripts/composer-setup.sh /tmp
RUN bash /tmp/composer-setup.sh

# APCU
RUN pear config-set http_proxy "${http_proxy}" \
    && pecl channel-update pecl.php.net \
    && pecl install apcu
RUN echo "extension=apcu.so\napc.enable_cli=on" > /usr/local/etc/php/conf.d/apcu.ini

# X-Debug
RUN pecl install xdebug-3.3.2 \
    && echo 'xdebug.mode=develop,debug,coverage' >> /usr/local/etc/php/php.ini \
    && echo 'xdebug.client_host=host.docker.internal' >> /usr/local/etc/php/php.ini \
    && docker-php-ext-enable xdebug

RUN mkdir -p var/cache var/log; \
	composer dump-autoload --classmap-authoritative --no-dev; \
	#composer dump-env prod; \
	composer run-script --no-dev post-install-cmd; \
	chmod +x bin/console; sync;

# Clean the mess
RUN apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Add Entrypoint
COPY ./automation/scripts/docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

RUN git config --global user.email "f.beaujois@gmail.com" \
    &&  git config --global user.name "fab5052"

# Run
CMD ["/usr/local/bin/docker-entrypoint.sh"]

