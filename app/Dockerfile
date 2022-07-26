FROM ubuntu:22.04

LABEL maintainer="Mack Knave"

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y gnupg tzdata \
    && echo "UTC" > /etc/timezone \
    && dpkg-reconfigure -f noninteractive tzdata

RUN apt-get update \
    && apt-get install -y curl zip unzip git supervisor sqlite3 \
       nginx php8.1-fpm php8.1-cli \
       php8.1-pgsql php8.1-sqlite3 php8.1-gd \
       php8.1-curl php8.1-memcached \
       php8.1-imap php8.1-mysql php8.1-mbstring \
       php8.1-xml php8.1-zip php8.1-bcmath php8.1-soap \
       php8.1-intl php8.1-readline php8.1-xdebug \
       php-msgpack php-igbinary \
    && php -r "readfile('http://getcomposer.org/installer');" | php -- --install-dir=/usr/bin/ --filename=composer \
#    && mkdir /run/php \
    && apt-get -y autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
    && echo -g "daemon off"; >> /etc/nginx/nginx.conf

ADD ./nginx/default /etc/nginx/sites-available/default
ADD ./process/supervisord.conf /etc/supervisor/conf.d/supervisord.conf

CMD ["supervisord"]

ADD ./php/php-fpm.conf /etc/php/8.1/fpm/php-fpm.conf

RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

#ADD ./scripts/start-container.sh /usr/bin/start-container.sh
#RUN chmod +x /usr/bin/start-container.sh
#
#ENTRYPOINT ["start-container"]