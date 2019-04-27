FROM phusion/baseimage:latest

RUN DEBIAN_FRONTEND=noninteractive
RUN locale-gen en_US.UTF-8

ENV LANGUAGE=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV LC_CTYPE=en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV TERM xterm

# Add the "PHP 7" ppa
RUN apt-get install -y software-properties-common && \
    add-apt-repository -y ppa:ondrej/php

#
#--------------------------------------------------------------------------
# Software's Installation
#--------------------------------------------------------------------------
#

# Install "PHP Extentions", "libraries", "Software's"
RUN apt-get update && \
    apt-get install -y --allow-downgrades --allow-remove-essential \
        --allow-change-held-packages \
        php7.2-cli \
        php7.2-common \
        php7.2-curl \
        php7.2-intl \
        php7.2-json \
        php7.2-xml \
        php7.2-mbstring \
        php7.2-mysql \
        php7.2-pgsql \
        php7.2-sqlite \
        php7.2-sqlite3 \
        php7.2-zip \
        php7.2-bcmath \
        php7.2-memcached \
        php7.2-gd \
        php7.2-dev \
        php7.2-xdebug \
        pkg-config \
        libcurl4-openssl-dev \
        libedit-dev \
        libssl-dev \
        libxml2-dev \
        xz-utils \
        libsqlite3-dev \
        sqlite3 \
        git \
        curl \
        vim \
        nano \
        postgresql-client \
        libpng16-16 \
        jpegoptim \
        optipng \
        pngquant \
        gifsicle \
        nasm

#####################################
# Composer:
#####################################

# Install composer and add its bin to the PATH.
RUN curl -s http://getcomposer.org/installer | php && \
    echo "export PATH=${PATH}:/var/www/vendor/bin" >> ~/.bashrc && \
    mv composer.phar /usr/local/bin/composer

# Source the bash
RUN . ~/.bashrc

# Start as root
USER root

###########################################################################
# ${USERID} non-root user:
###########################################################################

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG USERID=aaj
ENV USERID ${USERID}

ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

# always run apt update when start and after add new source list, then clean up at end.
RUN pecl channel-update pecl.php.net && \
    groupadd -g ${PGID} ${USERID} && \
    useradd -u ${PUID} -g ${USERID} -m ${USERID} -G docker_env && \
    usermod -p "*" ${USERID}

#
#--------------------------------------------------------------------------
# Mandatory Software's Installation
#--------------------------------------------------------------------------
#
# Mandatory Software's such as ("php-cli", "git", "vim", ....) are
# installed on the base image '${USERID}/workspace' image. If you want
# to add more Software's or remove existing one, you need to edit the
# base image (https://github.com/${USERID}/workspace).
#

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

ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

###########################################################################
# User Aliases
###########################################################################

COPY ./aliases.sh /root/aliases.sh
COPY ./aliases.sh /home/${USERID}/aliases.sh

RUN sed -i 's/\r//' /root/aliases.sh && \
    sed -i 's/\r//' /home/${USERID}/aliases.sh && \
    chown ${USERID}:${USERID} /home/${USERID}/aliases.sh && \
    echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc

###########################################################################
# Composer:
###########################################################################

USER root

# Add the composer.json
COPY ./composer.json /home/${USERID}/.composer/composer.json

# Make sure that ~/.composer belongs to ${USERID}
RUN chown -R ${USERID}:${USERID} /home/${USERID}/.composer


###########################################################################
# Non-root user : PHPUnit path
###########################################################################

# add ./vendor/bin to non-root user's bashrc (needed for phpunit)

# RUN echo "" >> ~/.bashrc && \
#     echo 'export PATH="/var/www/vendor/bin:$PATH"' >> ~/.bashrc

###########################################################################
# Crontab
###########################################################################

COPY ./crontab /etc/cron.d
RUN chmod -R 644 /etc/cron.d && \
    sed -i -e 's/USERID/${USERID}/g' /etc/cron.d/default

###########################################################################
# xDebug:
###########################################################################
RUN sed -i 's/^;//g' /etc/php/7.2/cli/conf.d/20-xdebug.ini && \
    echo "alias phpunit='php -dzend_extension=xdebug.so /var/www/vendor/bin/phpunit'" >> ~/.bashrc

# ADD for REMOTE debugging
COPY ./xdebug.ini /etc/php/7.2/cli/conf.d/xdebug.ini

RUN sed -i "s/xdebug.remote_autostart=0/xdebug.remote_autostart=1/" /etc/php/7.2/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.remote_enable=0/xdebug.remote_enable=1/" /etc/php/7.2/cli/conf.d/xdebug.ini && \
    sed -i "s/xdebug.cli_color=0/xdebug.cli_color=1/" /etc/php/7.2/cli/conf.d/xdebug.ini

###########################################################################
# PHP REDIS EXTENSION
###########################################################################
RUN printf "\n" | pecl -q install -o -f redis && \
    echo "extension=redis.so" >> /etc/php/7.2/mods-available/redis.ini && \
    phpenmod redis

###########################################################################
# Image optimizers:
###########################################################################
RUN exec bash && . ~/.bashrc && npm install -g svgo


USER ${USERID}

RUN composer global install

# Export composer vendor path
RUN echo "" >> ~/.bashrc && \
    echo 'export PATH="~/.composer/vendor/bin:$PATH"' >> ~/.bashrc

RUN echo "" >> ~/.bashrc && \
    echo "# Load Custom Aliases" >> ~/.bashrc && \
    echo "source ~/aliases.sh" >> ~/.bashrc && \
	echo "" >> ~/.bashrc

###########################################################################
# Node / NVM:
###########################################################################

# Check if NVM needs to be installed
ARG NODE_VERSION=node
ENV NODE_VERSION ${NODE_VERSION}
ENV NVM_DIR /home/${USERID}/.nvm

RUN mkdir -p $NVM_DIR && \
    curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.11/install.sh | bash \
        && . $NVM_DIR/nvm.sh \
        && nvm install ${NODE_VERSION} \
        && nvm use ${NODE_VERSION} \
        && nvm alias ${NODE_VERSION} \
        && npm install -g @vue/cli \
        && ln -s `npm bin --global` /home/${USERID}/.node-bin

# Wouldn't execute when added to the RUN statement in the above block
# Source NVM when loading bash since ~/.profile isn't loaded on non-login shell
RUN echo "" >> ~/.bashrc && \
    echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc && \
    echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> ~/.bashrc

# Add PATH for node
ENV PATH $PATH:/home/${USERID}/.node-bin

###########################################################################
# Prestissimo:
###########################################################################
RUN composer global require "hirak/prestissimo"


###########################################################################
# Check PHP version:
###########################################################################
RUN php -v | head -n 1 | grep -q "PHP 7.2."

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
