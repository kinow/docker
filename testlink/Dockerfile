FROM php:7.0-apache

RUN apt-get update

RUN apt-get -y install locales
RUN echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
RUN echo "fr_FR.UTF-8 UTF-8" >> /etc/locale.gen
RUN echo "en_GB.UTF-8 UTF-8" >> /etc/locale.gen
RUN echo "fi_FI.UTF-8 UTF-8" >> /etc/locale.gen
RUN echo "sv_SE.UTF-8 UTF-8" >> /etc/locale.gen
RUN locale-gen

RUN  apt-get install -y \
	zlib1g-dev \
	libpng-dev

RUN docker-php-ext-install mysqli \
	&& docker-php-ext-install mbstring \
	&& docker-php-ext-install zip \
	&& docker-php-ext-install json \
	&& docker-php-ext-install gd \
	&& docker-php-ext-install gettext

RUN a2enmod rewrite

RUN mkdir -p /var/www/html/gui/templates_c \
	&& mkdir -p /var/testlink/logs/ \
	&& mkdir -p /var/testlink/upload_area/ \
	&& chown -R www-data: /var/www/html/gui/templates_c \
	&& chown -R www-data: /var/testlink/

RUN echo "session.gc_maxlifetime=60000" >> /usr/local/etc/php/php.ini \
	&& echo "max_execution_time=3000" >> /usr/local/etc/php/php.ini
