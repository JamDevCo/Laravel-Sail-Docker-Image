FROM ubuntu:22.04

LABEL maintainer="Oshane Bailey"

ARG WWWGROUP=1000

WORKDIR /var/www/html

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC

# Set the correct timezone
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Install base dependencies and add the Ondřej Surý PPA for PHP
RUN apt-get update \
    && apt-get install -y software-properties-common curl gnupg2 \
    && add-apt-repository ppa:ondrej/php -y \
    && apt-get update

# Install PHP 8.1 and required extensions
RUN apt-get install -y php8.1-cli php8.1-dev cron \
    php8.1-pgsql php8.1-sqlite3 php8.1-gd \
    php8.1-curl php8.1-memcached \
    php8.1-imap php8.1-mysql php8.1-mbstring \
    php8.1-xml php8.1-zip php8.1-bcmath php8.1-soap \
    php8.1-intl php8.1-readline php8.1-pcov \
    php8.1-msgpack php8.1-igbinary php8.1-ldap \
    php8.1-redis php8.1-swoole php8.1-xdebug \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/bin --filename=composer

# Install Node.js and Yarn
RUN curl -sL https://deb.nodesource.com/setup_16.x | bash - \
    && apt-get install -y nodejs \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update && apt-get install -y yarn

# Install database clients
RUN apt-get install -y mysql-client postgresql-client

# Set capabilities for PHP binary
RUN setcap "cap_net_bind_service=+ep" /usr/bin/php8.1

# Create a user group and user
RUN groupadd --force -g $WWWGROUP sail \
    && useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u 1000 sail

# Copy custom scripts and configurations
COPY start-container /usr/local/bin/start-container
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY php.ini /etc/php/8.1/cli/conf.d/99-sail.ini
RUN chmod +x /usr/local/bin/start-container

# Expose application port
EXPOSE 8000

ENTRYPOINT ["start-container"]
