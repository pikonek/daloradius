# Official daloRADIUS Dockerfile for freeradius
# GitHub: https://github.com/lirantal/daloradius
#
# Build image:
# 1. git pull git@github.com:lirantal/daloradius.git
# 2. docker build -t lirantal/daloradius -f Dockerfile-freeradius
#
# Run the container:
# 1. docker run -p 80:80 -d lirantal/dalofreeradius

FROM freeradius/freeradius-server:latest
MAINTAINER Pikonek <dev.pisokonek@gmail.com>

LABEL Description="freeradius Docker based on Ubuntu 20.04 LTS, optimized for daloRADIUS." \
	License="GPLv2" \
	Usage="docker build -t lirantal/dalofreeradius -f Dockerfile-freeradius && docker run -d -p 80:80 lirantal/dalofreeradius" \
	Version="1.0"

ENV DEBIAN_FRONTEND noninteractive

# default timezone
ENV TZ Asia/Manila

RUN apt-get update \
	&& apt-get install --yes --no-install-recommends \
		apt-utils \
		ipcalc \
		freeradius-utils \
		apache2 \
		libapache2-mod-php \
		cron \
		net-tools \
		php \
		php-common \
		php-gd \
		php-curl \
		php-mail \
		php-dev \
		php-mail-mime \
		php-db \
		php-mysql \
		tzdata \
		net-tools \
		mariadb-client \
		libmysqlclient-dev \
		unzip \
		wget \
	&& rm -rf /var/lib/apt/lists/*

# Create directories
# /data should be mounted as volume to avoid recreation of database entries
RUN mkdir /app /data /internal_data
# WORKDIR /app

# PHP Pear DB library install
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
 && update-ca-certificates -f \
 && mkdir -p /tmp/pear/cache \
 && wget http://pear.php.net/go-pear.phar \
 && php go-pear.phar \
 && rm go-pear.phar \
 && pear channel-update pear.php.net \
 && pear install -a -f DB \
 && pear install -a -f Mail \
 && pear install -a -f Mail_Mime

ADD . /var/www/html
RUN touch /var/www/html/library/daloradius.conf.php
RUN chown -R www-data:www-data /var/www/html

# Enable the .htaccess in /var/www/html
RUN /bin/sed -i 's/AllowOverride\ None/AllowOverride\ All/g' /etc/apache2/apache2.conf

# Make init.sh script executable
RUN chmod +x /var/www/html/docker/init.sh

# Remove the original sample index.html file
RUN rm -rf /var/www/html/index.html

# Create daloRADIUS Log file
RUN touch /tmp/daloradius.log && chown -R www-data:www-data /tmp/daloradius.log

# Create freeradius log
# RUN mkdir /var/log/freeradius && touch /var/log/freeradius/radius.log

# Expose FreeRADIUS Ports and daloradius
EXPOSE 1812 1813 80

# Run the script which executes freeradius in foreground
CMD ["/var/www/html/docker/init.sh"]
ENTRYPOINT ["/var/www/html/docker/init.sh"]
