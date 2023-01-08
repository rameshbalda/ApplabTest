FROM ubuntu:latest
LABEL Author="Ramesh Balada" Description="A comprehensive docker image to run Apache-2.4 PHP-8.1"


# Stop dpkg-reconfigure tzdata from prompting for input
ENV DEBIAN_FRONTEND=noninteractive

# Install apache and php8
RUN apt update && \
    apt -y install \
        nano \
#        vim \
        sudo \
        curl \
        apache2 \
        php \
        php-fpm \
        libapache2-mod-php \
        libapache2-mod-auth-openidc \
        php-cli \
        php-curl \
        php-xml \
        php-mbstring \
        php-gd \
        php-mysql \
        php-json \
        php-common \
        php-ldap \
        php-memcached \
        php-mime-type \
        php-pgsql \
        php-tidy \
        php-intl \
        php-bcmath \
        php-xmlrpc \
        php-soap \
        php-uploadprogress \
        php-zip \
        php-pear \
        nodejs \
        npm \
        ssl-cert \
        git \
        git-flow \
# Ensure apache can bind to 80 as non-root
        libcap2-bin && \
    setcap 'cap_net_bind_service=+ep' /usr/sbin/apache2 && \
    dpkg --purge libcap2-bin && \
    apt -y autoremove && \
# As apache is never run as root, change dir ownership
    a2disconf other-vhosts-access-log && \
    chown -Rh www-data. /var/run/apache2 && \
# Install ImageMagick CLI tools
    apt -y install --no-install-recommends imagemagick && \
# Clean up apt setup files
    apt clean && \
    rm -rf /var/lib/apt/lists/* && \
# Setup apache
    a2enmod rewrite headers expires ext_filter ssl

# Override default apache and php config
COPY src/000-default.conf /etc/apache2/sites-available
COPY src/mpm_prefork.conf /etc/apache2/mods-available
COPY src/status.conf      /etc/apache2/mods-available
COPY src/99-local.ini     /etc/php/7.4/apache2/conf.d

RUN chown -R www-data:www-data /var
RUN chown -R www-data:www-data /etc/ssl
RUN chown -R www-data:www-data  /var/www/html

RUN chmod 2775  /var/www/html
RUN rm -f /var/www/html/index.html && \
    mkdir /var/www/html/.config && \
    tar cf /var/www/html/.config/etc-apache2.tar etc/apache2 && \
    tar cf /var/www/html/.config/etc-php.tar     etc/php && \
    dpkg -l > /var/www/html/.config/dpkg-l.txt

RUN export GIT_SSL_NO_VERIFY=1

#Composer setup
RUN export COMPOSER_HOME=/var/www/html/.composer/vendor/bin/
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
RUN php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
RUN php composer-setup.php
RUN php -r "unlink('composer-setup.php');"
RUN mv /composer.phar /var/www/html
RUN export COMPOSER_ALLOW_SUPERUSER=1 
ENV COMPOSER_ALLOW_SUPERUSER 1 

RUN git clone https://github.com/rameshbalda/ApplabTest.git /var/www/html/applab
RUN mv /var/www/html/composer.phar /var/www/html/applab/
WORKDIR /var/www/html/applab
RUN  /var/www/html/applab/composer.phar  install
#RUN  /var/www/html/composer.phar create-project drupal/recommended-project /var/www/html/applab

# format for chpasswd user_name:password
RUN echo "www-data:www-data" | chpasswd
RUN adduser www-data sudo

# Setup Apache2 HTTPS env
RUN a2ensite 000-default.conf

USER www-data

# Work directory

EXPOSE 80
EXPOSE 443
CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
