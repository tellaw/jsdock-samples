# Image : php:8.0.7-apache
# docker build . -t tellaw-php-80-apache
FROM php:8.0.7-apache

RUN apt-get update --fix-missing
RUN apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng-dev \
        libssl-dev \
        default-mysql-client \
        git \
        libzip-dev \
        zip \
        unzip \
        nano \
        vim \
        wget \
        gnupg \
        sudo \
        libxslt-dev \
        libxml2-dev \
	    apt-utils \
	    build-essential \
        bash-completion iproute2 \
    && docker-php-ext-install -j$(nproc) iconv \
    && docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install -j$(nproc) calendar

RUN apt-get install apt-transport-https

# install necessary locales (pour MS SQL...)
RUN apt-get install -y locales \
 && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
 && locale-gen

# NodeJs
RUN curl -sL https://deb.nodesource.com/setup_12.x | bash - \
    && apt-get install -y nodejs

# Yarn (pour Webpack Encore)
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update && apt-get install -y yarn

# Probe BLACKFIRE
#RUN version=$(php -r "echo PHP_MAJOR_VERSION.PHP_MINOR_VERSION;") \
#    && curl -A "Docker" -o /tmp/blackfire-probe.tar.gz -D - -L -s https://blackfire.io/api/v1/releases/probe/php/linux/amd64/$version \
#    && tar zxpf /tmp/blackfire-probe.tar.gz -C /tmp \
#    && mv /tmp/blackfire-*.so $(php -r "echo ini_get('extension_dir');")/blackfire.so \
#    && printf "extension=blackfire.so\nblackfire.agent_socket=tcp://blackfire:8707\n" > $PHP_INI_DIR/conf.d/blackfire.ini

# EXTENSION : LDAP
RUN apt-get install libldap2-dev -y && \
    rm -rf /var/lib/apt/lists/* && \
    docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu/ && \
    docker-php-ext-install ldap

RUN docker-php-ext-install intl pdo pdo_mysql zip exif xsl xml

# Extension pour Mongo client PHP
# Drivers mongo : https://docs.mongodb.com/ecosystem/drivers/driver-compatibility-reference/
# https://pecl.php.net/package/mongodb/1.4.0
RUN pecl install mongodb && docker-php-ext-enable mongodb

# Redis extension
RUN pecl install -o -f redis \
 &&  rm -rf /tmp/pear \
 &&  docker-php-ext-enable redis

# stomp pour ActiveMQ (commentaires actu WK)
# (NON COMPATIBLE PHP 8 ) RUN printf "no\n" | pecl install stomp && docker-php-ext-enable stomp

# Extension pour les Wordpress
RUN docker-php-ext-install mysqli && docker-php-ext-enable mysqli

# MS SQL tools and drivers
RUN curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
RUN curl https://packages.microsoft.com/config/debian/9/prod.list | tee /etc/apt/sources.list.d/mssql-tools.list
RUN apt-get update
RUN ACCEPT_EULA=Y apt-get install -y mssql-tools \
    unixodbc-dev
RUN echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile \
 && echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
RUN /bin/bash -c "source ~/.bashrc"

# PHP Driver pour SQL Server
RUN pecl install sqlsrv pdo_sqlsrv \
 && docker-php-ext-enable sqlsrv pdo_sqlsrv

# Composer
#RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin/ --filename=composer
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

RUN echo "date.timezone = 'Europe/Paris'" >> /usr/local/etc/php/conf.d/docker-php-custom-php.ini \
 && echo "memory_limit = '-1'" >> /usr/local/etc/php/conf.d/docker-php-custom-php.ini \
 && echo "log_errors = 'On'" >> /usr/local/etc/php/conf.d/docker-php-custom-php.ini \
 && echo "error_reporting = 'E_ALL'" >> /usr/local/etc/php/conf.d/docker-php-custom-php.ini \
 && echo "max_execution_time = '3600'" >> /usr/local/etc/php/conf.d/docker-php-custom-php.ini

 # Enable apache mods
RUN a2enmod rewrite remoteip expires headers

# Ajout variable environnement
ENV APACHE_DOCUMENT_ROOT /var/www/html
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf
RUN sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

RUN echo "<h1>Hello Container</h1>" > /var/www/html/index.html

# Mount VHost to : 
# docker run -p '80:80' --env 'APACHE_DOCUMENT_ROOT:/var/www/html' --name tellaw-test1-74-apache -d tellaw-php-74-apache
# './web/apache2/sites-available/000-default:/etc/apache2/sites-available/000-default'