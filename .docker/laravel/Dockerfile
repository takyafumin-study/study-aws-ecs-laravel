FROM php:8.3-fpm

ARG UNAME=www-data
ARG UGROUP=www-data
ARG UID=1000
ARG GID=1000
RUN usermod  --uid $UID $UNAME
RUN groupmod --gid $GID $UGROUP

COPY ./src /application

WORKDIR /application

COPY --from=composer /usr/bin/composer /usr/bin/composer

RUN apt-get update && apt-get install -y \
  git \
  zip \
  unzip \
  && composer install

RUN php artisan cache:clear \
  && php artisan config:clear \
  && php artisan route:clear \
  && php artisan view:clear

RUN chown -R www-data:www-data storage

USER ${UNAME}:${UGROUP}

