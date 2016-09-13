FROM debian:jessie

MAINTAINER keopx <keopx@keopx.net>

#
# Step 1: Installation
#

# Set frontend. We'll clean this later on!
ENV DEBIAN_FRONTEND noninteractive

ENV LOCALE es_ES.UTF-8

# Set repositories
RUN \
  echo "deb http://ftp.de.debian.org/debian/ jessie main non-free contrib\n" > /etc/apt/sources.list && \
  echo "deb-src http://ftp.de.debian.org/debian/ jessie main non-free contrib\n" >> /etc/apt/sources.list && \
  echo "deb http://security.debian.org/ jessie/updates main contrib non-free\n" >> /etc/apt/sources.list && \
  echo "deb-src http://security.debian.org/ jessie/updates main contrib non-free" >> /etc/apt/sources.list

# Update repositories cache and distribution
RUN apt-get -qq update && apt-get -qqy upgrade

# Install some basic tools needed for deployment
RUN apt-get -yqq install \
  apt-utils \
  build-essential \
  debconf-utils \
  debconf \
  locales \
  curl \
  wget \
  unzip \
  patch \
  rsync \
  vim \
  openssh-client \
  git-core \
  zsh \
  mysql-client

# Configure Dotdeb sources
RUN \
  wget -O - http://www.dotdeb.org/dotdeb.gpg | apt-key add - && \
  echo "deb http://packages.dotdeb.org jessie all\n" > /etc/apt/sources.list.d/dotdeb.list && \
  echo "deb-src http://packages.dotdeb.org jessie all" >> /etc/apt/sources.list.d/dotdeb.list && \
  apt-get -qq update


# Install PHP7 with Xdebug (dev environment)
RUN apt-get -yqq install \
  php7.0 \
  php7.0-curl \
  php7.0-dev \
  php7.0-gd \
  php7.0-intl \
  php7.0-json \
  php7.0-mcrypt \
  php7.0-mysql \
  libapache2-mod-php7.0

# Install image magick
RUN apt-get -yqq install imagemagick

# Install Apache web server
RUN apt-get -yqq install apache2-mpm-prefork

# Install ssmtp MTA
RUN apt-get -yqq install ssmtp

# Install PHP7 APCu
RUN apt-get -yqq install php7.0-apcu

# Install memcached service
RUN apt-get -yqq install memcached php7.0-memcached

# Install memcached service
RUN apt-get -yqq install php7.0-xml

#
# Step 2: Configuration
#

#
# PHP7 environment
#

# Development
RUN apt-get -yqq install php7.0-xdebug
#COPY config/php7/php.ini-development /etc/php/7.0/apache2/php.ini-development
# Production
COPY config/php7/php.ini-production /etc/php/7.0/apache2/php.ini

# Add apache vhosts
ADD config/vhosts/*.conf /etc/apache2/sites-available/

# Add apache web server vhost file
RUN rm /etc/apache2/sites-enabled/* && ln -s /etc/apache2/sites-available/* /etc/apache2/sites-enabled/

# Add apache conf
ADD config/apache2-conf/project.conf /etc/apache2/conf-available/

# Add apache web server configuration file
RUN ln -s /etc/apache2/conf-available/project.conf /etc/apache2/conf-enabled/project.conf

# Disable default ssl site
RUN a2dissite default-ssl

# Configure needed apache modules and disable default site
RUN a2enmod \
  access_compat \
  actions \
  alias \
  auth_basic \
  authn_core \
  authn_file \
  authz_core \
  authz_groupfile \
  authz_host \
  authz_user \
  autoindex \
  dir \
  env \
  expires \
  filter \
  mime \
  negotiation \
  php7.0 \
  mpm_prefork \
  reqtimeout \
  rewrite \
  setenvif \
  status \
  && a2dismod cgi

# Install composer (latest version)
RUN curl -sS https://getcomposer.org/installer | php && mv composer.phar /usr/local/bin/composer

### Install DRUSH (latest stable) ###
# Run this in your terminal to get the latest project version:
RUN curl http://files.drush.org/drush.phar -L -o drush.phar
# Accessing from anywhere on your system:
RUN mv drush.phar /usr/local/bin/drush
# Apply executable permissions on the downloaded file:
RUN chmod +x /usr/local/bin/drush
# Copy configuration files to user home directory:
RUN drush init -y

### DRUSH Faster way:
#RUN composer global require drush/drush:@stable

### Install DRUPAL CONSOLE (latest version) ###
# Run this in your terminal to get the latest project version:
RUN curl https://drupalconsole.com/installer -L -o drupal.phar
# Accessing from anywhere on your system:
RUN mv drupal.phar /usr/local/bin/drupal
# Apply executable permissions on the downloaded file:
RUN chmod +x /usr/local/bin/drupal
# Copy configuration files to user home directory:
RUN drupal init --override

### DRUPAL CONSOLE Faster way:
#RUN composer global require drupal/console:@stable

ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV APACHE_RUN_DIR /var/run/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2
RUN ln -sf /dev/stdout /var/log/apache2/access.log && \
    ln -sf /dev/stderr /var/log/apache2/error.log
RUN mkdir -p $APACHE_RUN_DIR $APACHE_LOCK_DIR $APACHE_LOG_DIR


RUN cp /etc/ssmtp/ssmtp.conf /etc/ssmtp/ssmtp.conf.bak
COPY ./config/ssmtp/ssmtp.conf /etc/ssmtp/ssmtp.conf


#
# Step 3: Clean the system
#

# Cleanup some things
RUN apt-get -q autoclean && \
  rm -rf /var/lib/apt/lists/*

#
# Step 4: Run
#

# Working dir
WORKDIR /var/www

# Volume for Apache2 data
VOLUME /var/www
RUN usermod -u 1000 www-data
RUN usermod -u 1000 www-data
RUN usermod -a -G users www-data
#RUN chown -R www-data:www-data /var/www


COPY scripts/apache2-foreground /usr/bin/

#RUN composer create-project drupal-composer/drupal-project:8.x-dev /var/www/unicef --stability dev --no-interaction
#RUN git clone git://github.com/robbyrussell/oh-my-zsh.git ~/.oh-my-zsh \
#      && cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc \
#      && chsh -s /bin/zsh


EXPOSE 80 443

CMD ["apache2-foreground"]

#CMD ["/bin/bash", "-c", " source /etc/apache2/envvars && exec /usr/sbin/apache2 -DFOREGROUND"]
