FROM composer:1.9.2 as composer

FROM php:7.4.2-cli-buster

# Add composer
COPY --from=composer /usr/bin/composer /usr/bin/composer

ENV DEBIAN_FRONTEND noninteractive

# Add a non-root user to prevent files being created with root permissions on host machine.
ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN set -xe; \
    apt-get update -yqq && \
    pecl channel-update pecl.php.net && \
    groupadd -g ${PGID} workspace && \
    useradd -u ${PUID} -g workspace -m workspace && \
    usermod -p "*" workspace -s /bin/bash && \
    apt-get install -yqq \
      apt-utils \
      cron \
      libzip-dev zip unzip && \
    docker-php-ext-install zip

# Set Timezone
ARG TZ=UTC
ENV TZ ${TZ}

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Add the composer.json
COPY ./composer.json /home/workspace/.composer/composer.json

# Add the auth.json for magento 2 credentials
COPY ./auth.json /home/workspace/.composer/auth.json

# Make sure that ~/.composer belongs to laradock
RUN chown -R workspace:workspace /home/workspace/.composer

USER workspace

ARG COMPOSER_PACKAGIST_URL
ENV COMPOSER_PACKAGIST_URL ${COMPOSER_PACKAGIST_URL}

RUN if [ ${COMPOSER_PACKAGIST_URL} ]; then \
    composer config -g repos.packagist composer ${COMPOSER_PACKAGIST_URL} \
;fi

# run the install
RUN composer global install

USER root

# crontab
COPY ./crontab /etc/cron.d

RUN chmod -R 644 /etc/cron.d
