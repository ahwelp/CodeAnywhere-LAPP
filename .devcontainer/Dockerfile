FROM php:7.4-apache

RUN mv "$PHP_INI_DIR/php.ini-development" "$PHP_INI_DIR/php.ini"
# Enable .htaccess files & mod_rewrite
RUN sed -i '/<Directory \/var\/www\/>/,/<\/Directory>/ s/AllowOverride None/AllowOverride all/' /etc/apache2/apache2.conf
RUN a2enmod rewrite

RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends git less ssh-client; rm -rf /var/lib/apt/lists/*

# install the PHP extensions, including XDebug
RUN set -ex; \	
	docker-php-ext-configure gd --with-freetype --with-jpeg; \
	docker-php-ext-install -j "$(nproc)" \		
		gd \
		postgresql \
		opcache \
		zip \
	; \	
  	pecl install xdebug-2.8.1; \
	docker-php-ext-enable xdebug; \
	rm -rf /var/lib/apt/lists/*

#XDebug settings
RUN echo "[XDebug]\nxdebug.remote_enable = 1\nxdebug.remote_autostart = 1" > $PHP_INI_DIR/conf.d/xdebug.ini

#install Composer
WORKDIR /tmp
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
		&& php composer-setup.php --quiet --install-dir=/usr/local/bin --filename=composer

#add vscode user
RUN useradd -ms /bin/bash vscode && usermod -aG www-data vscode

WORKDIR /var/www/html
USER www-data

USER root
RUN chown -R www-data:www-data /var/www/html/
RUN chmod g+w -R /var/www/html/
RUN find /var/www/html/ -type d -exec chmod g+s {} \;