FROM debian:stretch

MAINTAINER Ondrej Zalesky <ondrej.zalesky@eman.cz>

ARG APP_NAME
ARG LICENSE_KEY

# Timezone
ENV TZ=Europe/Prague

RUN \
  apt-get update && \
  apt-get install -y \
  wget gnupg apt-transport-https && \
  apt-get autoremove -y && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists 

RUN \
  wget -q https://packages.sury.org/php/apt.gpg -O- | apt-key add - && \
  echo "deb https://packages.sury.org/php/ stretch main" | tee /etc/apt/sources.list.d/php.list

RUN \
  apt-get update && \
  apt-get install -y \
    curl \
    unzip \
    vim \
    git \
    apt-transport-https \
    locales \
    iptables \
    apache2 \
    php5.6 \
    php5.6-mysql \
    php5.6-mcrypt \
    php5.6-mbstring \
    php5.6-gd \
    php5.6-memcache \
    php5.6-zip \
    php5.6-curl \
    php-pear \
    php5.6-apcu \
    php5.6-xml \
    php5.6-pgsql \
    php5.6-cli \
    php5.6-curl \
    php5.6-mcrypt \
    php5.6-sqlite \
    php5.6-intl \
    php5.6-tidy \
    php5.6-imap \
    php5.6-json \
    php5.6-imagick \
    php5.6-common \
    libapache2-mod-php5.6 \
    libapache2-mod-rpaf && \
  a2enmod proxy && \
  a2enmod proxy_http && \
  a2enmod alias && \
  a2enmod dir && \
  a2enmod env && \
  a2enmod mime && \
  a2enmod reqtimeout && \
  a2enmod rewrite && \
  a2enmod status && \
  a2enmod filter && \
  a2enmod deflate && \
  a2enmod setenvif && \
  a2enmod vhost_alias && \
  a2enmod ssl && \
  a2enmod php5.6 && \
  apt-get autoremove -y && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
  && dpkg-reconfigure --frontend noninteractive tzdata \
  && echo 'en_US.UTF-8 UTF-8\ncs_CZ.UTF-8 UTF-8\n' >> /etc/locale.gen \
  && /usr/sbin/locale-gen

RUN ln -sf /dev/stderr /var/log/apache2/error.log

#COPY ./docker/000-default.conf /etc/apache2/sites-enabled/000-default.conf
#COPY ./docker/run.sh /run.sh

#RUN chmod +x /run.sh

#EXPOSE 80

# copy config files
COPY config/sshd_config /etc/ssh/
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY config/entrypoint.sh /

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]

