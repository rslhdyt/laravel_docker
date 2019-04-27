#
#--------------------------------------------------------------------------
# Image Setup
#--------------------------------------------------------------------------
#
# To edit the 'php-fpm' base Image, visit its repository on Github
#    https://github.com/Laradock/php-fpm
#
# To change its version, see the available Tags on the Docker Hub:
#    https://hub.docker.com/r/laradock/php-fpm/tags/
#
# Note: Base Image name format {image-tag}-{php-version}
#

FROM php:7.2-fpm

RUN apt-get update \
  && apt-get install -y --no-install-recommends \
    curl \
    libmemcached-dev \
    libz-dev \
    libpq-dev \
    libjpeg-dev \
    libpng-dev \
    libfreetype6-dev \
    libssl-dev \
    libmcrypt-dev

RUN docker-php-ext-install pdo_mysql \
  # Install the PHP pdo_pgsql extention
  && docker-php-ext-install pdo_pgsql \
  # Install the PHP gd library
  && docker-php-ext-configure gd \
    --with-jpeg-dir=/usr/lib \
    --with-freetype-dir=/usr/include/freetype2 && \
  docker-php-ext-install gd

# Set Environment Variables
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update -yqq && \
    apt-get install -y apt-utils && \
    pecl channel-update pecl.php.net

#
#--------------------------------------------------------------------------
# Mandatory Software's Installation
#--------------------------------------------------------------------------
#
# Mandatory Software's such as ("mcrypt", "pdo_mysql", "libssl-dev", ....)
# are installed on the base image 'laradock/php-fpm' image. If you want
# to add more Software's or remove existing one, you need to edit the
# base image (https://github.com/Laradock/php-fpm).
#

#
#--------------------------------------------------------------------------
# Optional Software's Installation
#--------------------------------------------------------------------------
#
# Optional Software's will only be installed if you set them to `true`
# in the `docker-compose.yml` before the build.
# Example:
#   - INSTALL_ZIP_ARCHIVE=true
#

USER root

###########################################################################
# PHP REDIS EXTENSION
###########################################################################

# Install Php Redis Extension
RUN printf "\n" | pecl install -o -f redis \
    &&  rm -rf /tmp/pear \
    &&  docker-php-ext-enable redis

###########################################################################
# ZipArchive:
###########################################################################
RUN apt-get install libzip-dev -y && \
    docker-php-ext-configure zip --with-libzip && \
    # Install the zip extension
    docker-php-ext-install zip

# Installs pcntl, helpful for running Horizon
RUN docker-php-ext-install pcntl

# Install the bcmath extension
RUN docker-php-ext-install bcmath

# RUN docker-php-ext-install opcache

# Copy opcache configration
# COPY ./opcache.ini /usr/local/etc/php/conf.d/opcache.ini

RUN docker-php-ext-install mysqli

###########################################################################
# Tokenizer Modifications:
###########################################################################
RUN docker-php-ext-install tokenizer

###########################################################################
# Human Language and Character Encoding Support:
###########################################################################
RUN apt-get install -y zlib1g-dev libicu-dev g++ && \
    docker-php-ext-configure intl && \
    docker-php-ext-install intl

###########################################################################
# Image optimizers:
###########################################################################

USER root

RUN apt-get install -y jpegoptim optipng pngquant gifsicle

###########################################################################
# ImageMagick:
###########################################################################

USER root

RUN apt-get install -y libmagickwand-dev imagemagick && \
    pecl install imagick && \
    docker-php-ext-enable imagick

###########################################################################
# Check PHP version:
###########################################################################

RUN php -v | head -n 1 | grep -q "PHP 7.2."

#
#--------------------------------------------------------------------------
# Final Touch
#--------------------------------------------------------------------------
#
COPY ./laravel.ini /usr/local/etc/php/conf.d
COPY ./xlaravel.pool.conf /usr/local/etc/php-fpm.d/

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

RUN usermod -u 1000 www-data

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000
