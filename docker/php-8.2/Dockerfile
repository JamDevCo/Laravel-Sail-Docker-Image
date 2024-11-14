FROM ubuntu:22.04

LABEL maintainer="Oshane Bailey"

# Set the default value for WWWGROUP to 1000 if not provided
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

# Install PHP 8.2 and required extensions
RUN apt-get install -y php8.2-cli php8.2-dev cron \
    php8.2-pgsql php8.2-sqlite3 php8.2-gd \
    php8.2-curl php8.2-memcached \
    php8.2-imap php8.2-mysql php8.2-mbstring \
    php8.2-xml php8.2-zip php8.2-bcmath php8.2-soap \
    php8.2-intl php8.2-readline php8.2-pcov \
    php8.2-msgpack php8.2-igbinary php8.2-ldap \
    php8.2-redis php8.2-swoole php8.2-xdebug \
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
RUN setcap "cap_net_bind_service=+ep" /usr/bin/php8.2

# Create a user group and user with the specified WWWGROUP (default 1000)
RUN groupadd --force -g $WWWGROUP sail \
    && useradd -ms /bin/bash --no-user-group -g $WWWGROUP -u 1000 sail

# Copy custom scripts and configurations
COPY start-container /usr/local/bin/start-container
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY php.ini /etc/php/8.2/cli/conf.d/99-sail.ini
RUN chmod +x /usr/local/bin/start-container

# Expose application port
EXPOSE 8000

ENTRYPOINT ["start-container"]
